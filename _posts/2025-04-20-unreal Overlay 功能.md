---
title: unreal Overlay 功能
date: 2025-04-20
categories:
- CG
tags: 
- Unreal
- C++
---
unreal4 要实现多Pass功能需要自己改动源码，unreal5.1开始的Overlay支持了类似的功能，作用是在基础渲染之前，整个Mesh部分ID的用一个Overlay材质渲染，能够提供额外的边缘高光或者是描边，如要移植到unreal4，或者改动，需要注意一下以下代码。

#### UMeshComponent

定义和实现

```c++
UCLASS(abstract, ShowCategories = (VirtualTexture), MinimalAPI)
class UMeshComponent : public UPrimitiveComponent
{
	...
	/** Translucent material to blend on top of this mesh. Mesh will be rendered twice - once with a base material and once with overlay material */
	UPROPERTY(EditAnywhere, BlueprintReadOnly, AdvancedDisplay, Category=Rendering)
	TObjectPtr<class UMaterialInterface> OverlayMaterial;

	/** The max draw distance for overlay material. A distance of 0 indicates that overlay will be culled using primitive max distance. */
	UPROPERTY(EditAnywhere, BlueprintReadOnly, AdvancedDisplay, Category=Rendering)
	float OverlayMaterialMaxDrawDistance;
	...


UMaterialInterface* UMeshComponent::GetOverlayMaterial() const
{
	if (OverlayMaterial)
	{ 
		return OverlayMaterial;
	}
	else
	{
		return GetDefaultOverlayMaterial();
	}
}

void UMeshComponent::SetOverlayMaterial(UMaterialInterface* NewOverlayMaterial)
{
	if (OverlayMaterial != NewOverlayMaterial)
	{
		OverlayMaterial = NewOverlayMaterial;
		// Precache PSOs again
		PrecachePSOs();
		MarkRenderStateDirty();
	}
}
```

‍

#### SkeletalMesh.cpp

各类型的FPrimitiveSceneProxy的绘制函数中增加定义和绘制Overlay的部分

```c++
FSkeletalMeshSceneProxy::FSkeletalMeshSceneProxy(const USkinnedMeshComponent* Component, FSkeletalMeshRenderData* InSkelMeshRenderData)
		:	FPrimitiveSceneProxy(Component, Component->GetSkinnedAsset()->GetFName())
		,	Owner(Component->GetOwner())
		,	MeshObject(Component->MeshObject)
		,	SkeletalMeshRenderData(InSkelMeshRenderData)
		,	SkeletalMeshForDebug(Component->GetSkinnedAsset())
		,	PhysicsAssetForDebug(Component->GetPhysicsAsset())
		,	OverlayMaterial(Component->GetOverlayMaterial())
		,	OverlayMaterialMaxDrawDistance(Component->GetOverlayMaterialMaxDrawDistance())
	...
	if (OverlayMaterial != nullptr)
	{
		if (!OverlayMaterial->CheckMaterialUsage_Concurrent(MATUSAGE_SkeletalMesh))
		{
			OverlayMaterial = UMaterial::GetDefaultMaterial(MD_Surface);
			UE_LOG(LogSkeletalMesh, Warning, TEXT("Overlay material with missing usage flag was applied to skeletal mesh %s"),	*Component->GetSkinnedAsset()->GetPathName());
		}
	}


void FSkeletalMeshSceneProxy::DrawStaticElements(FStaticPrimitiveDrawInterface* PDI)
{
	if (!MeshObject)
	{
		return;
	}

	if (!HasViewDependentDPG())
	{
		ESceneDepthPriorityGroup PrimitiveDPG = GetStaticDepthPriorityGroup();
		bool bUseSelectedMaterial = false;

		int32 NumLODs = SkeletalMeshRenderData->LODRenderData.Num();
		int32 ClampedMinLOD = 0; // TODO: MinLOD, Bias?

		for (int32 LODIndex = ClampedMinLOD; LODIndex < NumLODs; ++LODIndex)
		{
			const FSkeletalMeshLODRenderData& LODData = SkeletalMeshRenderData->LODRenderData[LODIndex];

			if (LODSections.Num() > 0 && LODData.GetNumVertices() > 0)
			{
				float ScreenSize = MeshObject->GetScreenSize(LODIndex);
				const FLODSectionElements& LODSection = LODSections[LODIndex];
				check(LODSection.SectionElements.Num() == LODData.RenderSections.Num());

				for (FSkeletalMeshSectionIter Iter(LODIndex, *MeshObject, LODData, LODSection); Iter; ++Iter)
				{
					const FSkelMeshRenderSection& Section = Iter.GetSection();
					const int32 SectionIndex = Iter.GetSectionElementIndex();
					const FVertexFactory* VertexFactory = MeshObject->GetStaticSkinVertexFactory(LODIndex, SectionIndex, ESkinVertexFactoryMode::Default);

					if (!VertexFactory)
					{
						// hide this part
						continue;
					}

					const FSectionElementInfo& SectionElementInfo = Iter.GetSectionElementInfo();

					// If hidden skip the draw
					if (MeshObject->IsMaterialHidden(LODIndex, SectionElementInfo.UseMaterialIndex) || Section.bDisabled)
					{
						continue;
					}

				#if WITH_EDITOR
					if (GIsEditor)
					{
						bUseSelectedMaterial = (MeshObject->SelectedEditorSection == SectionIndex);
						PDI->SetHitProxy(SectionElementInfo.HitProxy);
					}
				#endif // WITH_EDITOR

					FMeshBatch MeshElement;
					FMeshBatchElement& BatchElement = MeshElement.Elements[0];
					MeshElement.DepthPriorityGroup = PrimitiveDPG;
					MeshElement.VertexFactory = VertexFactory;
					MeshElement.MaterialRenderProxy = SectionElementInfo.Material->GetRenderProxy();
					MeshElement.ReverseCulling = IsLocalToWorldDeterminantNegative();
					MeshElement.CastShadow = SectionElementInfo.bEnableShadowCasting;
				#if RHI_RAYTRACING
					MeshElement.CastRayTracedShadow = MeshElement.CastShadow && bCastDynamicShadow;
				#endif
					MeshElement.Type = PT_TriangleList;
					MeshElement.LODIndex = LODIndex;
					MeshElement.SegmentIndex = SectionIndex;
					MeshElement.MeshIdInPrimitive = SectionIndex;

					BatchElement.PrimitiveUniformBuffer = GetUniformBuffer();
					BatchElement.FirstIndex = Section.BaseIndex;
					BatchElement.MinVertexIndex = Section.BaseVertexIndex;
					BatchElement.MaxVertexIndex = LODData.GetNumVertices() - 1;
					BatchElement.NumPrimitives = Section.NumTriangles;
					BatchElement.IndexBuffer = LODData.MultiSizeIndexContainer.GetIndexBuffer();

					PDI->DrawMesh(MeshElement, ScreenSize);

					if (OverlayMaterial != nullptr)
					{
						FMeshBatch OverlayMeshBatch(MeshElement);
						OverlayMeshBatch.bOverlayMaterial = true;
						OverlayMeshBatch.CastShadow = false;
						OverlayMeshBatch.bSelectable = false;
						OverlayMeshBatch.MaterialRenderProxy = OverlayMaterial->GetRenderProxy();
						// make sure overlay is always rendered on top of base mesh
						OverlayMeshBatch.MeshIdInPrimitive += LODData.RenderSections.Num();
						// Reuse mesh ScreenSize as cull distance for an overlay. Overlay does not need to compute LOD so we can avoid adding new members into MeshBatch or MeshRelevance
						float OverlayMeshScreenSize = OverlayMaterialMaxDrawDistance;
						PDI->DrawMesh(OverlayMeshBatch, OverlayMeshScreenSize);
					}
				}
			}
		}
	}
}
```

#### SkinnedMeshComponent

```c++
protected:
	/** Get the default overlay material used by a mesh */
	ENGINE_API virtual UMaterialInterface* GetDefaultOverlayMaterial() const override;

	/** Get the default overlay material max draw distance */
	ENGINE_API virtual float GetDefaultOverlayMaterialMaxDrawDistance() const override;

void USkinnedMeshComponent::CollectPSOPrecacheData(const FPSOPrecacheParams& BasePrecachePSOParams, FMaterialInterfacePSOPrecacheParamsList& OutParams)

void USkinnedMeshComponent::GetUsedMaterials( TArray<UMaterialInterface*>& OutMaterials, bool bGetDebugMaterials ) const
```

#### StaticMeshRender.cpp

```c++
FStaticMeshSceneProxy::FStaticMeshSceneProxy(const FStaticMeshSceneProxyDesc& InProxyDesc, bool bForceLODsShareStaticLighting)
	: FPrimitiveSceneProxy(InProxyDesc, InProxyDesc.GetStaticMesh()->GetFName())
	, RenderData(InProxyDesc.GetStaticMesh()->GetRenderData())
	, OverlayMaterial(InProxyDesc.GetOverlayMaterial())
	, OverlayMaterialMaxDrawDistance(InProxyDesc.GetOverlayMaterialMaxDrawDistance())

void FStaticMeshSceneProxy::DrawStaticElements(FStaticPrimitiveDrawInterface* PDI)

void FStaticMeshSceneProxyDesc::InitializeFromStaticMeshComponent(const UStaticMeshComponent* InComponent)
```

#### StaticMeshComponent.cpp

```c++
void UStaticMeshComponent::CollectPSOPrecacheDataImpl(
	const FVertexFactoryType* VFType, 
	const FPSOPrecacheParams& BasePrecachePSOParams, 
	GetPSOVertexElementsFn GetVertexElements,
	FMaterialInterfacePSOPrecacheParamsList& OutParams) const

void UStaticMeshComponent::GetUsedMaterials(TArray<UMaterialInterface*>& OutMaterials, bool bGetDebugMaterials) const
```

#### InstancedStaticMesh.cpp

```c++
void FInstancedStaticMeshSceneProxy::GetDynamicMeshElements(const TArray<const FSceneView*>& Views, const FSceneViewFamily& ViewFamily, uint32 VisibilityMap, FMeshElementCollector& Collector) const

void FInstancedStaticMeshSceneProxy::SetupProxy(const FInstancedStaticMeshSceneProxyDesc& InProxyDesc)
```

#### HierarchicalInstancedStaticMesh.cpp

```c++
void FHierarchicalStaticMeshSceneProxy::FillDynamicMeshElements(const FSceneView* View, FMeshElementCollector& Collector, const FFoliageElementParams& ElementParams, const FFoliageRenderInstanceParams& Params) const
```

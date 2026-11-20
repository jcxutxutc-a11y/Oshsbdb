-- سكربت مسح الجودة والخامات + منع الانعكاس نهائياً مع بقاء نفس ألوان الماب
local Lighting = game:GetService("Lighting")
local Terrain = workspace:FindFirstChildOfClass("Terrain")
local MaterialService = game:GetService("MaterialService")

-- 1. منع الانعكاس واللمعان والظلال من جذور الإضاءة
Lighting.GlobalShadows = false
Lighting.FogEnd = 9e9
Lighting.EnvironmentDiffuseScale = 0
Lighting.EnvironmentSpecularScale = 0
Lighting.OutdoorAmbient = Lighting.Ambient -- توحيد الإضاءة لمنع تدرجات اللمعان

-- إزالة السماء (Sky) أو استبدالها بسماء سادة يمنع انعكاس الغيوم والشمس على المجسمات
for _, v in pairs(Lighting:GetChildren()) do
    if v.Name == "DimmedLook" or v:IsA("Sky") then
        v:Destroy()
    elseif v:IsA("BloomEffect") or v:IsA("SunRaysEffect") or v:IsA("DepthOfFieldEffect") or v:IsA("Atmosphere") or v:IsA("BlurEffect") then
        v.Enabled = false
    end
end

-- إجبار نظام الإضاءة على الوضع الكلاسيكي البسيط بدون انعكاسات (Compatibility / Voxel)
pcall(function()
    sethiddenproperty(Lighting, "Technology", Enum.Technology.Compatibility)
end)

-- 2. إيقاف الخامات المخصصة في الماب
pcall(function()
    MaterialService.Use2022Materials = false
    for _, v in pairs(MaterialService:GetDescendants()) do
        if v:IsA("MaterialVariant") then
            v:Destroy()
        end
    end
end)

-- 3. تنعيم التضاريس ومنع انعكاس المياه
if Terrain then
    Terrain.WaterWaveSize = 0
    Terrain.WaterWaveSpeed = 0
    Terrain.WaterReflectance = 0
    Terrain.WaterTransparency = 0
    Terrain.Decoration = false
end

-- 4. مسح الجودة والخامات ومنع الانعكاس عن كل المجسمات
local function stripQualityAndReflection(obj)
    if obj:IsA("BasePart") then
        obj.CastShadow = false
        obj.Reflectance = 0
        -- SmoothPlastic مع تقنية Compatibility وبدون سماء يعطي لوناً مطفياً 100% بدون أي لمعة
        if obj.Material ~= Enum.Material.Neon then
            obj.Material = Enum.Material.SmoothPlastic
        end
    elseif obj:IsA("Decal") or obj:IsA("Texture") or obj:IsA("SurfaceAppearance") then
        obj:Destroy()
    elseif obj:IsA("MeshPart") then
        obj.TextureID = ""
        obj.Reflectance = 0
        if obj.Material ~= Enum.Material.Neon then
            obj.Material = Enum.Material.SmoothPlastic
        end
    elseif obj:IsA("SpecialMesh") then
        obj.TextureId = ""
    elseif obj:IsA("ParticleEmitter") or obj:IsA("Trail") then
        obj.Lifetime = NumberRange.new(0.1, 0.2)
    end
end

-- تطبيق على الماب بالكامل
for _, item in pairs(workspace:GetDescendants()) do
    pcall(stripQualityAndReflection, item)
end

-- تطبيق مستمر على أي مجسم جديد
workspace.DescendantAdded:Connect(function(item)
    pcall(stripQualityAndReflection, item)
end)

-- 5. إجبار المحرك على أقل جودة رسومية
settings().Rendering.QualityLevel = Enum.QualityLevel.Level01

game:GetService("StarterGui"):SetCore("SendNotification", {
    Title = "تم تفعيل وضع بدون انعكاس",
    Text = "تم مسح الجودة وإلغاء جميع الانعكاسات بنجاح!",
    Duration = 5
})

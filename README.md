-- دالة لحذف مؤثرات النار والانفجارات والدخان فقط
local function clearEffects(obj)
    if obj:IsA("Fire") or obj:IsA("Smoke") or obj:IsA("Explosion") or obj:IsA("ParticleEmitter") or obj:IsA("Sparkles") or obj:IsA("Trail") or obj:IsA("Beam") then
        obj:Destroy()
    end
end

-- مسح المؤثرات الموجودة حالياً في الماب
for _, v in ipairs(workspace:GetDescendants()) do
    pcall(clearEffects, v)
end

-- مسح أي نار أو انفجار أو دخان جديد يظهر تلقائياً أثناء اللعب
workspace.DescendantAdded:Connect(function(v)
    task.wait()
    pcall(clearEffects, v)
end)

print("تم مسح النار والانفجارات والدخان مع الإبقاء على الخامات!")

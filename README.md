loadstring(game:HttpGet("local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local Workspace = game:GetService("Workspace")

local player = Players.LocalPlayer
local camera = Workspace.CurrentCamera

-- إعدادات الطيران الأساسية
local speed = 50 
local flying = false
local bg, bv

-- ==========================================
-- إنشاء واجهة المستخدم (مناسبة للهاتف)
-- ==========================================
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "MobileFlyGui"
-- استخدام CoreGui لضمان عدم اختفاء اللوحة عند الموت
ScreenGui.Parent = game:GetService("CoreGui") 
ScreenGui.ResetOnSpawn = false

-- الإطار الرئيسي الذي يجمع الأزرار
local MainFrame = Instance.new("Frame")
MainFrame.Parent = ScreenGui
MainFrame.Size = UDim2.new(0, 180, 0, 90)
MainFrame.Position = UDim2.new(0, 15, 0, 15) -- الموقع أعلى اليسار
MainFrame.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
MainFrame.BorderSizePixel = 2
MainFrame.BorderColor3 = Color3.fromRGB(0, 170, 255)
MainFrame.Active = true
MainFrame.Draggable = true -- جعل اللوحة قابلة للسحب لتحريكها في الشاشة

-- زر تشغيل وإيقاف الطيران
local ToggleBtn = Instance.new("TextButton")
ToggleBtn.Parent = MainFrame
ToggleBtn.Size = UDim2.new(1, -10, 0, 35)
ToggleBtn.Position = UDim2.new(0, 5, 0, 5)
ToggleBtn.BackgroundColor3 = Color3.fromRGB(60, 60, 60)
ToggleBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
ToggleBtn.Text = "الطيران: متوقف"
ToggleBtn.Font = Enum.Font.GothamBold
ToggleBtn.TextSize = 14

-- زر تقليل السرعة (-)
local MinusBtn = Instance.new("TextButton")
MinusBtn.Parent = MainFrame
MinusBtn.Size = UDim2.new(0, 40, 0, 40)
MinusBtn.Position = UDim2.new(0, 5, 0, 45)
MinusBtn.BackgroundColor3 = Color3.fromRGB(200, 50, 50)
MinusBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
MinusBtn.Text = "-"
MinusBtn.Font = Enum.Font.GothamBold
MinusBtn.TextSize = 24

-- نص عرض السرعة الحالية
local SpeedLabel = Instance.new("TextLabel")
SpeedLabel.Parent = MainFrame
SpeedLabel.Size = UDim2.new(1, -100, 0, 40)
SpeedLabel.Position = UDim2.new(0, 50, 0, 45)
SpeedLabel.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
SpeedLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
SpeedLabel.Text = "السرعة: " .. speed
SpeedLabel.Font = Enum.Font.GothamBold
SpeedLabel.TextSize = 14

-- زر زيادة السرعة (+)
local PlusBtn = Instance.new("TextButton")
PlusBtn.Parent = MainFrame
PlusBtn.Size = UDim2.new(0, 40, 0, 40)
PlusBtn.Position = UDim2.new(1, -45, 0, 45)
PlusBtn.BackgroundColor3 = Color3.fromRGB(50, 200, 50)
PlusBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
PlusBtn.Text = "+"
PlusBtn.Font = Enum.Font.GothamBold
PlusBtn.TextSize = 24

-- ==========================================
-- برمجة أزرار السرعة
-- ==========================================
PlusBtn.MouseButton1Click:Connect(function()
    speed = speed + 10 -- زيادة السرعة بمقدار 10
    SpeedLabel.Text = "السرعة: " .. speed
end)

MinusBtn.MouseButton1Click:Connect(function()
    if speed > 10 then -- منع السرعة من أن تصبح بالسالب أو صفر
        speed = speed - 10
        SpeedLabel.Text = "السرعة: " .. speed
    end
end)

-- ==========================================
-- نظام الطيران والتوجيه
-- ==========================================
local function ToggleFly()
    local character = player.Character or player.CharacterAdded:Wait()
    local humanoid = character:WaitForChild("Humanoid")
    local rootPart = character:WaitForChild("HumanoidRootPart")

    flying = not flying

    if flying then
        ToggleBtn.Text = "الطيران: يعمل"
        ToggleBtn.BackgroundColor3 = Color3.fromRGB(0, 150, 0)
        
        humanoid.PlatformStand = true

        bg = Instance.new("BodyGyro", rootPart)
        bg.P = 9e4
        bg.maxTorque = Vector3.new(9e9, 9e9, 9e9)
        bg.cframe = rootPart.CFrame

        bv = Instance.new("BodyVelocity", rootPart)
        bv.velocity = Vector3.new(0, 0, 0)
        bv.maxForce = Vector3.new(9e9, 9e9, 9e9)

        _G.FlyLoop = RunService.RenderStepped:Connect(function()
            if not flying then return end

            local moveDir = humanoid.MoveDirection

            if moveDir.Magnitude > 0 then
                -- توجيه الحركة لتتبع الكاميرا
                local localMove = camera.CFrame:VectorToObjectSpace(moveDir)
                local flyVector = camera.CFrame:VectorToWorldSpace(Vector3.new(localMove.X, 0, localMove.Z))
                
                bv.Velocity = flyVector.Unit * speed
            else
                bv.Velocity = Vector3.new(0, 0, 0)
            end
            
            -- تعديل دوران الشخصية لتنظر حيث تنظر الكاميرا
            bg.CFrame = CFrame.new(camera.CFrame.Position, camera.CFrame.Position + camera.CFrame.LookVector)
        end)
    else
        ToggleBtn.Text = "الطيران: متوقف"
        ToggleBtn.BackgroundColor3 = Color3.fromRGB(60, 60, 60)
        
        humanoid.PlatformStand = false
        
        if bg then bg:Destroy() end
        if bv then bv:Destroy() end
        if _G.FlyLoop then _G.FlyLoop:Disconnect() end
    end
end

-- ربط زر التشغيل بدالة الطيران
ToggleBtn.MouseButton1Click:Connect(ToggleFly)
"))()

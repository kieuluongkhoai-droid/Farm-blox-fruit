-- Script Blox Fruit: Rainbow UI + Teleport (Customizable) + Auto Quest
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local LocalPlayer = Players.LocalPlayer

-- Cài đặt mặc định
local Config = {
    Distance = 12,
    Speed = 6,
    Height = 2
}

local CommF = ReplicatedStorage:FindFirstChild("Remotes") and ReplicatedStorage.Remotes:FindFirstChild("CommF_")

-- Tạo Giao Diện (UI)
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "RainbowBloxFruitGui"
ScreenGui.Parent = game.CoreGui

local MainFrame = Instance.new("Frame")
MainFrame.Size = UDim2.new(0, 320, 0, 420) -- Tăng chiều cao để chứa thêm ô nhập liệu
MainFrame.Position = UDim2.new(0.5, -160, 0.5, -210)
MainFrame.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
MainFrame.BorderSizePixel = 0
MainFrame.Active = true
MainFrame.Draggable = true
MainFrame.Parent = ScreenGui

local MainCorner = Instance.new("UICorner")
MainCorner.CornerRadius = UDim.new(0, 10)
MainCorner.Parent = MainFrame

local UIStroke = Instance.new("UIStroke")
UIStroke.Thickness = 3
UIStroke.Parent = MainFrame

task.spawn(function()
    local hue = 0
    while true do
        hue = (hue + 1) % 360
        UIStroke.Color = Color3.fromHSV(hue / 360, 1, 1)
        task.wait(0.03)
    end
end)

-- Hàm tạo Nút/ô nhập
local function CreateButton(text, pos, callback)
    local btn = Instance.new("TextButton")
    btn.Size = UDim2.new(0.85, 0, 0, 35)
    btn.Position = pos
    btn.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
    btn.Text = text
    btn.TextColor3 = Color3.fromRGB(255, 255, 255)
    btn.Parent = MainFrame
    Instance.new("UICorner", btn).CornerRadius = UDim.new(0, 8)
    btn.MouseButton1Click:Connect(callback)
    return btn
end

-- Hàm tạo Ô nhập liệu
local function CreateInput(placeholder, pos, configKey)
    local box = Instance.new("TextBox")
    box.Size = UDim2.new(0.85, 0, 0, 35)
    box.Position = pos
    box.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
    box.TextColor3 = Color3.fromRGB(255, 255, 255)
    box.Text = tostring(Config[configKey])
    box.PlaceholderText = placeholder
    box.Parent = MainFrame
    Instance.new("UICorner", box).CornerRadius = UDim.new(0, 8)
    
    box.FocusLost:Connect(function()
        local num = tonumber(box.Text)
        if num then Config[configKey] = num end
    end)
    return box
end

-- UI Controls
CreateButton("Auto Quest: TẮT", UDim2.new(0.075, 0, 0.05, 0), function(btn)
    autoQuestActive = not autoQuestActive
    btn.Text = autoQuestActive and "Auto Quest: BẬT" or "Auto Quest: TẮT"
    btn.BackgroundColor3 = autoQuestActive and Color3.fromRGB(0, 170, 0) or Color3.fromRGB(50, 50, 50)
end)

CreateButton("Mob Teleport: TẮT", UDim2.new(0.075, 0, 0.15, 0), function(btn)
    teleportActive = not teleportActive
    btn.Text = teleportActive and "Mob Teleport: BẬT" or "Mob Teleport: TẮT"
    btn.BackgroundColor3 = teleportActive and Color3.fromRGB(0, 170, 0) or Color3.fromRGB(50, 50, 50)
end)

local ModeBtn = CreateButton("Chế độ: Dưới chân", UDim2.new(0.075, 0, 0.25, 0), function()
    currentMode = currentMode % 3 + 1
    ModeBtn.Text = "Chế độ: " .. modes[currentMode]
end)

-- Ô nhập liệu
CreateInput("Khoảng cách (Default: 12)", UDim2.new(0.075, 0, 0.35, 0), "Distance")
CreateInput("Tốc độ xoay (Default: 6)", UDim2.new(0.075, 0, 0.45, 0), "Speed")
CreateInput("Độ cao (Default: 2)", UDim2.new(0.075, 0, 0.55, 0), "Height")

-- Biến logic
local autoQuestActive = false
local teleportActive = false
local currentMode = 1
local modes = {"Dưới chân", "Trên đầu", "Bay quanh"}

-- Logic xử lý chính
task.spawn(function()
    while true do
        if autoQuestActive and CommF then
            pcall(function()
                local questUI = LocalPlayer.PlayerGui:FindFirstChild("Main") and LocalPlayer.PlayerGui.Main:FindFirstChild("Quest")
                if not questUI or not questUI.Visible then
                    CommF:InvokeServer("RequestQuest")
                end
            end)
        end
        task.wait(2)
    end
end)

RunService.RenderStepped:Connect(function()
    if not teleportActive then return end
    local character = LocalPlayer.Character
    if not character or not character:FindFirstChild("HumanoidRootPart") then return end
    local rootPart = character.HumanoidRootPart

    local enemies = workspace:FindFirstChild("Enemies")
    if enemies then
        local angle = tick() * Config.Speed -- Tốc độ xoay tùy chỉnh
        local count = 0
        
        for _, enemy in ipairs(enemies:GetChildren()) do
            local enemyRoot = enemy:FindFirstChild("HumanoidRootPart")
            local humanoid = enemy:FindFirstChild("Humanoid")
            if enemyRoot and humanoid and humanoid.Health > 0 then
                count = count + 1
                if currentMode == 1 then
                    enemyRoot.CFrame = rootPart.CFrame + Vector3.new(0, -Config.Height, 0)
                elseif currentMode == 2 then
                    enemyRoot.CFrame = rootPart.CFrame + Vector3.new(0, Config.Height, 0)
                elseif currentMode == 3 then
                    local radius = Config.Distance -- Khoảng cách tùy chỉnh
                    local theta = angle + (count * (math.pi * 2 / 6))
                    local x = math.cos(theta) * radius
                    local z = math.sin(theta) * radius
                    enemyRoot.CFrame = CFrame.new(rootPart.Position + Vector3.new(x, Config.Height, z))
                end
            end
        end
    end
end)

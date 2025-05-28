local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local RunService = game:GetService("RunService")
local Workspace = game:GetService("Workspace")

local player = Players.LocalPlayer
local camera = Workspace.CurrentCamera

local function getCharacter()
    return player.Character or player.CharacterAdded:Wait()
end

local function getHRP()
    local char = getCharacter()
    return char:WaitForChild("HumanoidRootPart")
end

-- UI Setup
local screenGui = Instance.new("ScreenGui", game.CoreGui)
screenGui.Name = "UltimateHackHub"
screenGui.ResetOnSpawn = false

local mainFrame = Instance.new("Frame", screenGui)
mainFrame.Size = UDim2.new(0, 350, 0, 380)
mainFrame.Position = UDim2.new(0.5, -175, 0.5, -190)
mainFrame.BackgroundColor3 = Color3.fromRGB(20,20,20)
mainFrame.Active = true
mainFrame.Draggable = true

local title = Instance.new("TextLabel", mainFrame)
title.Size = UDim2.new(1, 0, 0, 35)
title.Text = "Ultimate Hack Hub"
title.BackgroundColor3 = Color3.fromRGB(0, 170, 140)
title.TextColor3 = Color3.new(1,1,1)
title.Font = Enum.Font.GothamBold
title.TextSize = 20

local buttonsFrame = Instance.new("ScrollingFrame", mainFrame)
buttonsFrame.Size = UDim2.new(1, -20, 1, -50)
buttonsFrame.Position = UDim2.new(0, 10, 0, 40)
buttonsFrame.BackgroundColor3 = Color3.fromRGB(35,35,35)
buttonsFrame.BorderSizePixel = 0
buttonsFrame.CanvasSize = UDim2.new(0,0,0,0)
buttonsFrame.ScrollBarThickness = 6

local UIListLayout = Instance.new("UIListLayout", buttonsFrame)
UIListLayout.Padding = UDim.new(0,8)
UIListLayout.SortOrder = Enum.SortOrder.LayoutOrder

-- =====================
-- Functional Scripts
-- =====================

local Scripts = {}

-- Fly
Scripts["Fly"] = function()
    local flying = false
    local speed = 60
    local bodyGyro, bodyVelocity

    local function start()
        local hrp = getHRP()
        local humanoid = getCharacter():FindFirstChildOfClass("Humanoid")
        if not hrp or not humanoid then return end

        bodyGyro = Instance.new("BodyGyro", hrp)
        bodyGyro.P = 9e4
        bodyGyro.MaxTorque = Vector3.new(9e9, 9e9, 9e9)
        bodyGyro.CFrame = hrp.CFrame

        bodyVelocity = Instance.new("BodyVelocity", hrp)
        bodyVelocity.MaxForce = Vector3.new(9e9,9e9,9e9)
        bodyVelocity.Velocity = Vector3.zero

        humanoid.PlatformStand = true
        flying = true
    end

    local function stop()
        flying = false
        local humanoid = getCharacter():FindFirstChildOfClass("Humanoid")
        if bodyGyro then bodyGyro:Destroy() end
        if bodyVelocity then bodyVelocity:Destroy() end
        if humanoid then humanoid.PlatformStand = false end
    end

    start()

    local conn = RunService.Heartbeat:Connect(function()
        if flying then
            local move = Vector3.zero
            if UserInputService:IsKeyDown(Enum.KeyCode.W) then move += camera.CFrame.LookVector end
            if UserInputService:IsKeyDown(Enum.KeyCode.S) then move -= camera.CFrame.LookVector end
            if UserInputService:IsKeyDown(Enum.KeyCode.A) then move -= camera.CFrame.RightVector end
            if UserInputService:IsKeyDown(Enum.KeyCode.D) then move += camera.CFrame.RightVector end
            if move.Magnitude > 0 then
                bodyVelocity.Velocity = move.Unit * speed
            else
                bodyVelocity.Velocity = Vector3.zero
            end
            bodyGyro.CFrame = camera.CFrame
        else
            conn:Disconnect()
        end
    end)

    -- Toggle key (E)
    UserInputService.InputBegan:Connect(function(input)
        if input.KeyCode == Enum.KeyCode.E then
            if flying then
                stop()
            else
                start()
            end
        end
    end)
end

-- Speed
Scripts["Speed"] = function()
    local humanoid = getCharacter():FindFirstChildOfClass("Humanoid")
    humanoid.WalkSpeed = 50
end

-- Noclip
Scripts["Noclip"] = function()
    local noclip = true
    RunService.Stepped:Connect(function()
        if noclip then
            for _, v in pairs(getCharacter():GetDescendants()) do
                if v:IsA("BasePart") and v.CanCollide == true then
                    v.CanCollide = false
                end
            end
        end
    end)
end

-- BTools (versão moderna)
Scripts["BTools"] = function()
    local backpack = player:FindFirstChildOfClass("Backpack")
    local tool = Instance.new("Tool", backpack)
    tool.RequiresHandle = false
    tool.Name = "Destroy"
    tool.Activated:Connect(function()
        local target = player:GetMouse().Target
        if target then
            target:Destroy()
        end
    end)
end

-- ESP
Scripts["ESP"] = function()
    local function createESP(plr)
        local esp = Instance.new("BillboardGui", plr.Character:WaitForChild("Head"))
        esp.Size = UDim2.new(4, 0, 1, 0)
        esp.Adornee = plr.Character.Head
        esp.AlwaysOnTop = true

        local frame = Instance.new("Frame", esp)
        frame.Size = UDim2.new(1,0,1,0)
        frame.BackgroundColor3 = Color3.new(1,0,0)
        frame.BackgroundTransparency = 0.3
        frame.BorderSizePixel = 0
    end

    for _,plr in pairs(Players:GetPlayers()) do
        if plr ~= player then
            createESP(plr)
        end
    end

    Players.PlayerAdded:Connect(function(plr)
        plr.CharacterAdded:Connect(function()
            wait(1)
            createESP(plr)
        end)
    end)
end

-- Aimlock
Scripts["Aimlock"] = function()
    local locking = false

    UserInputService.InputBegan:Connect(function(input)
        if input.KeyCode == Enum.KeyCode.Q then
            locking = not locking
        end
    end)

    RunService.RenderStepped:Connect(function()
        if locking then
            local closest = nil
            local shortest = math.huge
            for _, plr in pairs(Players:GetPlayers()) do
                if plr ~= player and plr.Character and plr.Character:FindFirstChild("Head") then
                    local pos = camera:WorldToViewportPoint(plr.Character.Head.Position)
                    local dist = (Vector2.new(pos.X, pos.Y) - UserInputService:GetMouseLocation()).Magnitude
                    if dist < shortest then
                        shortest = dist
                        closest = plr
                    end
                end
            end
            if closest then
                camera.CFrame = CFrame.new(camera.CFrame.Position, closest.Character.Head.Position)
            end
        end
    end)
end

-- =====================
-- UI Button Generator
-- =====================

local function createButton(name, func)
    local btn = Instance.new("TextButton", buttonsFrame)
    btn.Size = UDim2.new(1, -20, 0, 40)
    btn.BackgroundColor3 = Color3.fromRGB(0, 170, 140)
    btn.BorderSizePixel = 0
    btn.Font = Enum.Font.GothamBold
    btn.TextSize = 18
    btn.TextColor3 = Color3.new(1, 1, 1)
    btn.Text = name

    btn.MouseButton1Click:Connect(function()
        pcall(func)
    end)
end

for name, func in pairs(Scripts) do
    createButton(name, func)
end

-- Ajustar CanvasSize
task.wait(0.2)
buttonsFrame.CanvasSize = UDim2.new(0, 0, 0, UIListLayout.AbsoluteContentSize.Y)

print("[Ultimate Hack Hub] Loaded com sucesso!")

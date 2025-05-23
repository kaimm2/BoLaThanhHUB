local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local RunService = game:GetService("RunService")

local LocalPlayer = Players.LocalPlayer
local Character = LocalPlayer.Character or LocalPlayer.CharacterAdded:Wait()
local Humanoid = Character:WaitForChild("Humanoid")

local Rayfield = loadstring(game:HttpGet('https://sirius.menu/rayfield'))()
local Window = Rayfield:CreateWindow({
    Name = "Rothanhpc",
    LoadingTitle = "Rothanhpc,",
    LoadingSubtitle = "VersionFREE-AimLock",
    Theme = "Dark"
})

local Tab = Window:CreateTab("Main", 4483362458)
local AntiStunToggle
local isHoldingW = false
local AntiStunLoop
local function TrackKeys()
    UserInputService.InputBegan:Connect(function(input, gameProcessed)
        if gameProcessed then return end
        if input.KeyCode == Enum.KeyCode.W then
            isHoldingW = true
        end
    end)
    UserInputService.InputEnded:Connect(function(input)
        if input.KeyCode == Enum.KeyCode.W then
            isHoldingW = false
        end
    end)
end

local function StartAntiStun()
    Character = LocalPlayer.Character or LocalPlayer.CharacterAdded:Wait()
    Humanoid = Character:WaitForChild("Humanoid")
    TrackKeys()
end

AntiStunToggle = Tab:CreateToggle({
    Name = "Anti Stun",
    CurrentValue = false,
    Callback = function(Value)
        if Value then
            StartAntiStun()
            AntiStunLoop = RunService.RenderStepped:Connect(function()
                if Humanoid then
                    if isHoldingW then
                        Humanoid.WalkSpeed = 27
                    else
                        Humanoid.WalkSpeed = 16
                    end
                end
            end)
        else
            if AntiStunLoop then AntiStunLoop:Disconnect() end
            if Humanoid then
                Humanoid.WalkSpeed = 16
            end
        end
    end,
})

LocalPlayer.CharacterAdded:Connect(function(newCharacter)
    Character = newCharacter
    if AntiStunToggle.CurrentValue then
        StartAntiStun()
    end
end)

local Workspace = game:GetService("Workspace")
local AntiRagdollToggle, FreezeOnLaunchToggle
local AntiRagdollLoop, AnchorLaunchLoop

AntiRagdollToggle = Tab:CreateToggle({
    Name = "Anti Ragdoll",
    CurrentValue = false,
    Callback = function(Value)
        if Value then
            AntiRagdollLoop = RunService.RenderStepped:Connect(function()
                local HumanoidRootPart = Character:FindFirstChild("HumanoidRootPart")
                local Ragdoll = Character:FindFirstChild("Ragdoll")
                if HumanoidRootPart then
                    HumanoidRootPart.Anchored = Ragdoll and Ragdoll:IsA("Accessory")
                end
            end)
        else
            if AntiRagdollLoop then AntiRagdollLoop:Disconnect() end
            local HumanoidRootPart = Character:FindFirstChild("HumanoidRootPart")
            if HumanoidRootPart then HumanoidRootPart.Anchored = false end
        end
    end,
})

FreezeOnLaunchToggle = Tab:CreateToggle({
    Name = "Anti Launch",
    CurrentValue = false,
    Callback = function(Value)
        if Value then
            AnchorLaunchLoop = RunService.RenderStepped:Connect(function()
                local HumanoidRootPart = Character:FindFirstChild("HumanoidRootPart")
                local BeingLaunched = Character:FindFirstChild("BeingLaunched")
                if HumanoidRootPart then
                    HumanoidRootPart.Anchored = BeingLaunched and BeingLaunched:IsA("Accessory")
                end
            end)
        else
            if AnchorLaunchLoop then AnchorLaunchLoop:Disconnect() end
            local HumanoidRootPart = Character:FindFirstChild("HumanoidRootPart")
            if HumanoidRootPart then HumanoidRootPart.Anchored = false end
        end
    end,
})


LocalPlayer.CharacterAdded:Connect(function(newCharacter)
    Character = Workspace.Live:WaitForChild(LocalPlayer.Name)
end)

local LockOnTab = Window:CreateTab("Lock-On", 4483362458)

local LockOnToggle, CameraLockToggle, LockLoop, CameraLockLoop, RangeSlider, SpeedSlider, DeadCheckToggle, VisualizerToggle
local MaxLockOnRange = 30
local LockOnSpeed = 0.1
local DisregardDead = false
local Visualizer, VisualizerConnection
local DiscRotation = Vector3.new(0, 0, 90)

LockOnToggle = LockOnTab:CreateToggle({
    Name = "Closest Player Lock-On",
    CurrentValue = false,
    Callback = function(Value)
        if Value then
            LockLoop = RunService.RenderStepped:Connect(function()
                local closest, closestDist = nil, math.huge
                for _, player in ipairs(Players:GetPlayers()) do
                    if player ~= LocalPlayer and player.Character and player.Character:FindFirstChild("HumanoidRootPart") then
                        local humanoid = player.Character:FindFirstChild("Humanoid")
                        local distance = (LocalPlayer.Character.PrimaryPart.Position - player.Character.HumanoidRootPart.Position).Magnitude
                        if distance < closestDist and distance <= MaxLockOnRange then
                            if DisregardDead and humanoid and humanoid.Health > 0 then
                                closest, closestDist = player, distance
                            elseif not DisregardDead then
                                closest, closestDist = player, distance
                            end
                        end
                    end
                end
                if closest and closest.Character and closest.Character:FindFirstChild("HumanoidRootPart") then
                    local chrPos = LocalPlayer.Character.PrimaryPart.Position
                    local targetPos = closest.Character.HumanoidRootPart.Position
                    local modPos = Vector3.new(targetPos.X, chrPos.Y, targetPos.Z)
                    local currentCFrame = LocalPlayer.Character.PrimaryPart.CFrame
                    local targetCFrame = CFrame.new(chrPos, modPos)
                    LocalPlayer.Character:SetPrimaryPartCFrame(currentCFrame:Lerp(targetCFrame, LockOnSpeed))
                end
            end)
        else
            if LockLoop then LockLoop:Disconnect() end
        end
    end,
})

CameraLockToggle = LockOnTab:CreateToggle({
    Name = "Camera Lock-On",
    CurrentValue = false,
    Callback = function(Value)
        if Value then
            CameraLockLoop = RunService.RenderStepped:Connect(function()
                local closest, closestDist = nil, math.huge
                for _, player in ipairs(Players:GetPlayers()) do
                    if player ~= LocalPlayer and player.Character and player.Character:FindFirstChild("HumanoidRootPart") then
                        local humanoid = player.Character:FindFirstChild("Humanoid")
                        local distance = (LocalPlayer.Character.PrimaryPart.Position - player.Character.HumanoidRootPart.Position).Magnitude
                        if distance < closestDist and distance <= MaxLockOnRange then
                            if DisregardDead and humanoid and humanoid.Health > 0 then
                                closest, closestDist = player, distance
                            elseif not DisregardDead then
                                closest, closestDist = player, distance
                            end
                        end
                    end
                end
                if closest and closest.Character and closest.Character:FindFirstChild("HumanoidRootPart") then
                    local cam = workspace.CurrentCamera
                    local camCFrame = cam.CFrame
                    local camTargetCFrame = CFrame.new(cam.CFrame.Position, closest.Character.HumanoidRootPart.Position)
                    cam.CFrame = camCFrame:Lerp(camTargetCFrame, LockOnSpeed)
                end
            end)
        else
            if CameraLockLoop then CameraLockLoop:Disconnect() end
        end
    end,
})

VisualizerToggle = LockOnTab:CreateToggle({
    Name = "Lock-On Range Visualizer",
    CurrentValue = false,
    Callback = function(Value)
        if Value then
            Visualizer = Instance.new("Part")
            Visualizer.Name = "VisualizerESP"
            Visualizer.Anchored = true
            Visualizer.CanCollide = false
            Visualizer.Size = Vector3.new(0, MaxLockOnRange * 2, MaxLockOnRange * 2)
            Visualizer.CFrame = CFrame.new(LocalPlayer.Character.HumanoidRootPart.Position + Vector3.new(0, -2.0, 0)) * CFrame.Angles(math.rad(DiscRotation.X), math.rad(DiscRotation.Y), math.rad(DiscRotation.Z))
            Visualizer.Material = Enum.Material.Neon
            Visualizer.Transparency = 0.95
            Visualizer.Color = Color3.new(0.258824, 1.000000, 0.258824)
            Visualizer.Shape = Enum.PartType.Cylinder
            Visualizer.Parent = workspace
            VisualizerConnection = RunService.RenderStepped:Connect(function()
                if LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart") then
                    Visualizer.Size = Vector3.new(0, MaxLockOnRange * 2, MaxLockOnRange * 2)
                    Visualizer.CFrame = CFrame.new(LocalPlayer.Character.HumanoidRootPart.Position + Vector3.new(0, -2.0, 0)) * CFrame.Angles(math.rad(DiscRotation.X), math.rad(DiscRotation.Y), math.rad(DiscRotation.Z))
                end
            end)
        else
            if Visualizer then
                Visualizer:Destroy()
                Visualizer = nil
            end
            if VisualizerConnection then
                VisualizerConnection:Disconnect()
                VisualizerConnection = nil
            end
        end
    end,
})

RangeSlider = LockOnTab:CreateSlider({
    Name = "Lock-On Range",
    Range = {10, 100},
    Increment = 1,
    Suffix = "Studs",
    CurrentValue = 30,
    Callback = function(Value)
        MaxLockOnRange = Value
    end,
})

SpeedSlider = LockOnTab:CreateSlider({
    Name = "Lock-On Speed/Smoothing ",
    Range = {0, 1.1},
    Increment = 0.01,
    Suffix = "",
    CurrentValue = 0.1,
    Callback = function(Value)
        LockOnSpeed = Value
    end,
})

DeadCheckToggle = LockOnTab:CreateToggle({
    Name = "Disregard Dead People",
    CurrentValue = false,
    Callback = function(Value)
        DisregardDead = Value
    end,
})
local ExtraTab = Window:CreateTab("Extra", 4483362458)

ExtraTab:CreateButton({
    Name = "Infinite Yield",
    Callback = function()
        loadstring(game:HttpGet('https://raw.githubusercontent.com/EdgeIY/infiniteyield/master/source'))()
    end
})

ExtraTab:CreateButton({
    Name = "Remote Spy",
    Callback = function()
        loadstring(game:HttpGet("https://raw.githubusercontent.com/infyiff/backup/main/SimpleSpyV3/main.lua"))()
    end
})

ExtraTab:CreateButton({
    Name = "Dex Explorer",
    Callback = function()
        loadstring(game:HttpGet("https://raw.githubusercontent.com/infyiff/backup/main/dex.lua"))()
    end
})

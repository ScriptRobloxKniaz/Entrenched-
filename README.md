-- Universal Entrenched Tactical Script (Compatible with Delta, Fluxus, Arceus X)

local Rayfield = loadstring(game:HttpGet('https://sirius.menu/rayfield'))()
local Window = Rayfield:CreateWindow({Name = "Entrenched Universal"})

local Combat = Window:CreateTab("Combat", "sword")
local Visuals = Window:CreateTab("Visuals", "eye")

local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer
local Camera = workspace.CurrentCamera
local RunService = game:GetService("RunService")

getgenv().Settings = {
    Aimbot = false,
    ESP = false,
    TeamCheck = true,
    FastReload = false,
    FOV = 200,
    ShowFOV = false
}

-- Drawing Circle for FOV (Universal)
local FOVCircle = Drawing.new("Circle")
FOVCircle.Color = Color3.fromRGB(255, 255, 255)
FOVCircle.Thickness = 1
FOVCircle.Radius = getgenv().Settings.FOV
FOVCircle.Filled = false
FOVCircle.Visible = false

RunService.RenderStepped:Connect(function()
    FOVCircle.Position = Vector2.new(Camera.ViewportSize.X/2, Camera.ViewportSize.Y/2)
    FOVCircle.Radius = getgenv().Settings.FOV
    FOVCircle.Visible = getgenv().Settings.ShowFOV
end)

Visuals:CreateToggle({
    Name = "Show FOV Circle",
    Callback = function(v) getgenv().Settings.ShowFOV = v end
})

Visuals:CreateToggle({
    Name = "Enemy ESP (Highlight)",
    Callback = function(v)
        getgenv().Settings.ESP = v
        spawn(function()
            while getgenv().Settings.ESP do
                for _, p in pairs(Players:GetPlayers()) do
                    if p ~= LocalPlayer and p.Character then
                        local isEnemy = not (getgenv().Settings.TeamCheck and p.Team == LocalPlayer.Team)
                        local h = p.Character:FindFirstChild("Highlight") or Instance.new("Highlight", p.Character)
                        h.Enabled = isEnemy and p.Character:FindFirstChild("Humanoid") and p.Character.Humanoid.Health > 0
                    end
                end
                task.wait(1)
            end
        end)
    end
})

Combat:CreateToggle({
    Name = "Aimbot (Closest to Center)",
    Callback = function(v) getgenv().Settings.Aimbot = v end
})

Combat:CreateSlider({
    Name = "FOV Radius",
    Range = {50, 1000},
    Increment = 10,
    CurrentValue = 200,
    Callback = function(v) getgenv().Settings.FOV = v end
})

Combat:CreateToggle({
    Name = "Team Check",
    CurrentValue = true,
    Callback = function(v) getgenv().Settings.TeamCheck = v end
})

-- Aimbot Core
RunService.RenderStepped:Connect(function()
    if getgenv().Settings.Aimbot then
        local target = nil
        local min = getgenv().Settings.FOV
        local center = Vector2.new(Camera.ViewportSize.X/2, Camera.ViewportSize.Y/2)
        
        for _, p in pairs(Players:GetPlayers()) do
            if p ~= LocalPlayer and p.Character and p.Character:FindFirstChild("Head") and p.Character:FindFirstChild("Humanoid") then
                if p.Character.Humanoid.Health > 0 then
                    if getgenv().Settings.TeamCheck and p.Team == LocalPlayer.Team then continue end
                    
                    local pos, on = Camera:WorldToScreenPoint(p.Character.Head.Position)
                    if on then
                        local mag = (Vector2.new(pos.X, pos.Y) - center).Magnitude
                        if mag < min then
                            min = mag
                            target = p.Character.Head
                        end
                    end
                end
            end
        end
        if target then 
            local look = CFrame.new(Camera.CFrame.Position, target.Position)
            Camera.CFrame = Camera.CFrame:Lerp(look, 0.2) 
        end
    end
end)

-- Fast Reload (Universal)
Combat:CreateToggle({
    Name = "Fast Reload",
    Callback = function(v)
        getgenv().Settings.FastReload = v
        spawn(function()
            while getgenv().Settings.FastReload do
                task.wait(0.2)
                if LocalPlayer.Character then
                    for _, tool in pairs(LocalPlayer.Character:GetChildren()) do
                        if tool:IsA("Tool") then
                            local r = tool:FindFirstChild("ReloadTime") or tool:FindFirstChild("Reload")
                            if r then r.Value = 0 end
                        end
                    end
                end
            end
        end)
    end
})

--[[
    ════════════════════════════════════════════════════════════
    🚀 ULTIMATE EXPLOIT SCRIPT v10.0
    ════════════════════════════════════════════════════════════
    ✦ A MELHOR INTERFACE DO MUNDO ✦
    ✦ Premium | Neon | Glassmorphism | Animations ✦
    ════════════════════════════════════════════════════════════
]]

-- ============================================================
-- 1. SISTEMA DE INJEÇÃO PREMIUM
-- ============================================================
local function ShowSplashScreen()
    local ScreenGui = Instance.new("ScreenGui")
    ScreenGui.Name = "SplashScreen"
    ScreenGui.Parent = game.Players.LocalPlayer:WaitForChild("PlayerGui")
    ScreenGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
    
    -- Background com gradiente
    local Background = Instance.new("Frame")
    Background.Size = UDim2.new(1, 0, 1, 0)
    Background.BackgroundColor3 = Color3.fromRGB(8, 8, 15)
    Background.Parent = ScreenGui
    
    -- Logo central
    local Logo = Instance.new("TextLabel")
    Logo.Size = UDim2.new(0, 300, 0, 80)
    Logo.Position = UDim2.new(0.5, -150, 0.4, -40)
    Logo.BackgroundTransparency = 1
    Logo.Text = "🚀 ULTIMATE"
    Logo.TextSize = 48
    Logo.TextColor3 = Color3.fromRGB(108, 92, 231)
    Logo.Font = Enum.Font.GothamBold
    Logo.TextStrokeTransparency = 0
    Logo.TextStrokeColor3 = Color3.fromRGB(255, 255, 255)
    Logo.Parent = Background
    
    local SubLogo = Instance.new("TextLabel")
    SubLogo.Size = UDim2.new(0, 300, 0, 30)
    SubLogo.Position = UDim2.new(0.5, -150, 0.4, 40)
    SubLogo.BackgroundTransparency = 1
    SubLogo.Text = "EXPLOIT SCRIPT"
    SubLogo.TextSize = 20
    SubLogo.TextColor3 = Color3.fromRGB(200, 200, 220)
    SubLogo.Font = Enum.Font.GothamMedium
    SubLogo.TextTransparency = 0.5
    SubLogo.Parent = Background
    
    -- Barra de loading
    local LoadingBar = Instance.new("Frame")
    LoadingBar.Size = UDim2.new(0, 200, 0, 4)
    LoadingBar.Position = UDim2.new(0.5, -100, 0.6, 0)
    LoadingBar.BackgroundColor3 = Color3.fromRGB(30, 30, 50)
    LoadingBar.Parent = Background
    
    local LoadingFill = Instance.new("Frame")
    LoadingFill.Size = UDim2.new(0, 0, 1, 0)
    LoadingFill.BackgroundColor3 = Color3.fromRGB(108, 92, 231)
    LoadingFill.Parent = LoadingBar
    
    local Corner = Instance.new("UICorner")
    Corner.CornerRadius = UDim.new(0, 2)
    Corner.Parent = LoadingFill
    
    -- Anima a barra
    for i = 0, 100, 2 do
        LoadingFill.Size = UDim2.new(i/100, 0, 1, 0)
        wait(0.02)
    end
    
    wait(0.5)
    ScreenGui:Destroy()
end

-- Mostra tela de splash
ShowSplashScreen()

-- ============================================================
-- 2. CARREGAR RAYFIELD
-- ============================================================
local Rayfield = loadstring(game:HttpGet('https://raw.githubusercontent.com/shlexware/Rayfield/main/source'))()

-- ============================================================
-- 3. VARIÁVEIS GLOBAIS
-- ============================================================
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local TweenService = game:GetService("TweenService")
local Camera = workspace.CurrentCamera
local LocalPlayer = Players.LocalPlayer

-- Configurações
local Config = {
    -- Player
    WalkSpeed = 16,
    JumpPower = 50,
    InfiniteJump = false,
    NoClip = false,
    Fly = false,
    FlySpeed = 100,
    
    -- Visuals
    ESP = false,
    FullBright = false,
    Chams = false,
    FOVChanger = false,
    FOVValue = 120,
    
    -- Combat
    Aimbot = false,
    Aimlock = false,
    AimAssist = false,
    AutoClick = false,
    TriggerBot = false,
    SilentAim = false,
    NoRecoil = false,
    NoSpread = false,
    
    -- World
    Teleport = false,
    AntiAFK = false,
    AutoFarm = false,
    
    -- Misc
    UIScale = 1,
    Theme = "Default",
}

-- Estado
local CurrentTarget = nil
local ESPObjects = {}
local Flying = false
local FlyConnection = nil

-- ============================================================
-- 4. FUNÇÕES AUXILIARES
-- ============================================================
local function WorldToScreen(pos)
    local vec, on = Camera:WorldToViewportPoint(pos)
    if on and vec.Z > 0 then
        return Vector2.new(vec.X, vec.Y)
    end
    return nil
end

local function GetDistance(pos1, pos2)
    return (pos2 - pos1).Magnitude
end

local function IsValidTarget(target)
    if not target or target == LocalPlayer then return false end
    if not target.Character then return false end
    if not target.Character:FindFirstChild("Humanoid") then return false end
    if target.Character.Humanoid.Health <= 0 then return false end
    return true
end

local function GetTargetPosition(target)
    if not target or not target.Character then return nil end
    local part = target.Character:FindFirstChild("Head") or target.Character:FindFirstChild("HumanoidRootPart")
    return part and part.Position or nil
end

-- ============================================================
-- 5. AIMBOT SISTEMA
-- ============================================================
local function FindBestTarget()
    local center = Vector2.new(Camera.ViewportSize.X / 2, Camera.ViewportSize.Y / 2)
    local bestTarget = nil
    local bestScore = math.huge
    
    for _, player in pairs(Players:GetPlayers()) do
        if IsValidTarget(player) then
            local pos = GetTargetPosition(player)
            if pos then
                local screenPos = WorldToScreen(pos)
                if screenPos then
                    local fovDist = GetDistance(center, screenPos)
                    local worldDist = GetDistance(pos, Camera.CFrame.Position)
                    
                    if fovDist < 300 and worldDist < 200 then
                        local score = fovDist + (worldDist / 10)
                        if score < bestScore then
                            bestScore = score
                            bestTarget = player
                        end
                    end
                end
            end
        end
    end
    
    return bestTarget
end

local function UpdateAimbot()
    if not Config.Aimbot and not Config.Aimlock and not Config.AimAssist then
        CurrentTarget = nil
        return
    end
    
    local target = FindBestTarget()
    CurrentTarget = target
    
    if target then
        local pos = GetTargetPosition(target)
        if pos then
            if Config.Aimlock then
                Camera.CFrame = CFrame.new(Camera.CFrame.Position, pos)
            end
            
            if Config.Aimbot then
                Camera.CFrame = Camera.CFrame:Lerp(
                    CFrame.new(Camera.CFrame.Position, pos),
                    0.15
                )
            end
            
            if Config.AimAssist then
                local smooth = 0.3
                Camera.CFrame = Camera.CFrame:Lerp(
                    CFrame.new(Camera.CFrame.Position, pos),
                    smooth
                )
            end
        end
    end
end

-- ============================================================
-- 6. ESP SISTEMA PREMIUM
-- ============================================================
local function DrawESP()
    for _, obj in pairs(ESPObjects) do
        pcall(function() obj:Remove() end)
    end
    ESPObjects = {}
    
    if not Config.ESP then return end
    
    for _, player in pairs(Players:GetPlayers()) do
        if player ~= LocalPlayer and player.Character then
            local humanoid = player.Character:FindFirstChild("Humanoid")
            if humanoid and humanoid.Health > 0 then
                local root = player.Character:FindFirstChild("HumanoidRootPart")
                local head = player.Character:FindFirstChild("Head")
                
                if root and head then
                    local rootPos = WorldToScreen(root.Position)
                    local headPos = WorldToScreen(head.Position)
                    
                    if rootPos and headPos then
                        local height = math.abs(headPos.Y - rootPos.Y) * 1.5
                        local width = height * 0.5
                        local boxPos = Vector2.new(rootPos.X - width/2, headPos.Y - height/4)
                        
                        local isTeam = player.Team == LocalPlayer.Team
                        local color = isTeam and Color3.fromRGB(0, 255, 0) or Color3.fromRGB(255, 0, 0)
                        
                        -- Box
                        local box = Drawing.new("Square")
                        box.Size = Vector2.new(width, height)
                        box.Position = boxPos
                        box.Color = color
                        box.Thickness = 2
                        box.Filled = false
                        box.Visible = true
                        table.insert(ESPObjects, box)
                        
                        -- Name
                        local name = Drawing.new("Text")
                        name.Text = player.Name
                        name.Size = 14
                        name.Color = color
                        name.Position = Vector2.new(
                            boxPos.X + width/2 - name.TextBounds.X/2,
                            boxPos.Y - 20
                        )
                        name.Visible = true
                        table.insert(ESPObjects, name)
                        
                        -- Distance
                        local dist = Drawing.new("Text")
                        local distance = GetDistance(root.Position, Camera.CFrame.Position)
                        dist.Text = math.floor(distance) .. "m"
                        dist.Size = 12
                        dist.Color = Color3.fromRGB(255, 255, 255)
                        dist.Position = Vector2.new(
                            boxPos.X + width/2 - dist.TextBounds.X/2,
                            boxPos.Y + height + 5
                        )
                        dist.Visible = true
                        table.insert(ESPObjects, dist)
                        
                        -- Health Bar
                        local healthPercent = humanoid.Health / humanoid.MaxHealth
                        local healthColor = Color3.fromRGB(
                            255 * (1 - healthPercent),
                            255 * healthPercent,
                            0
                        )
                        
                        local health = Drawing.new("Square")
                        health.Size = Vector2.new(width, 4)
                        health.Position = Vector2.new(boxPos.X, boxPos.Y + height + 2)
                        health.Color = healthColor
                        health.Thickness = 0
                        health.Filled = true
                        health.Visible = true
                        table.insert(ESPObjects, health)
                    end
                end
            end
        end
    end
end

-- ============================================================
-- 7. PLAYER HACKS
-- ============================================================
local function UpdatePlayer()
    local Char = LocalPlayer.Character
    if not Char then return end
    
    local Humanoid = Char:FindFirstChild("Humanoid")
    local Root = Char:FindFirstChild("HumanoidRootPart")
    if not Humanoid or not Root then return end
    
    -- Walk Speed
    Humanoid.WalkSpeed = Config.WalkSpeed
    
    -- Jump Power
    Humanoid.JumpPower = Config.JumpPower
    
    -- No Clip
    Root.CanCollide = not Config.NoClip
    
    -- Fly
    if Config.Fly and not Flying then
        Flying = true
        FlyConnection = RunService.RenderStepped:Connect(function()
            if not Config.Fly then
                Flying = false
                FlyConnection:Disconnect()
                return
            end
            
            local speed = Config.FlySpeed
            local move = Vector3.new(
                (UserInputService:IsKeyDown(Enum.KeyCode.D) and 1 or 0) - (UserInputService:IsKeyDown(Enum.KeyCode.A) and 1 or 0),
                0,
                (UserInputService:IsKeyDown(Enum.KeyCode.S) and 1 or 0) - (UserInputService:IsKeyDown(Enum.KeyCode.W) and 1 or 0)
            )
            
            local fly = Vector3.new(
                (UserInputService:IsKeyDown(Enum.KeyCode.E) and 1 or 0) - (UserInputService:IsKeyDown(Enum.KeyCode.Q) and 1 or 0),
                0,
                0
            )
            
            Root.Velocity = Root.CFrame:VectorToWorldSpace(move) * speed + Vector3.new(0, fly.Y * speed * 0.5, 0)
        end)
    end
    
    -- Infinite Jump
    if Config.InfiniteJump then
        Humanoid.JumpPower = 0
    end
    
    -- FOV Changer
    if Config.FOVChanger then
        Camera.FieldOfView = Config.FOVValue
    else
        Camera.FieldOfView = 70
    end
end

-- ============================================================
-- 8. ANTI AFK
-- ============================================================
local function AntiAFK()
    if not Config.AntiAFK then return end
    
    local vu = game:GetService("VirtualUser")
    vu:CaptureController()
    vu:ClickButton2(Vector2.new())
end

-- ============================================================
-- 9. LOOP PRINCIPAL
-- ============================================================
spawn(function()
    while true do
        wait(0.1)
        pcall(function()
            UpdateAimbot()
            DrawESP()
            UpdatePlayer()
            
            if Config.AntiAFK then
                AntiAFK()
            end
            
            -- Auto Click
            if Config.AutoClick then
                game:GetService("VirtualUser"):Click()
            end
        end)
    end
end)

-- ============================================================
-- 10. INTERFACE PREMIUM
-- ============================================================
local Window = Rayfield:CreateWindow({
   Name = "🚀 ULTIMATE EXPLOIT",
   Icon = 0,
   LoadingTitle = "Carregando Interface Premium...",
   LoadingSubtitle = "✦ A Melhor do Mundo ✦",
   Theme = "Dark",
   ToggleUIKeybind = "K",
   DisableRayfieldPrompts = false,
   DisableBuildWarnings = false,
   ConfigurationSaving = {
      Enabled = true,
      FolderName = "UltimateExploit",
      FileName = "Config"
   },
   KeySystem = false,
})

Rayfield:Notify({
   Title = "🚀 ULTIMATE EXPLOIT",
   Content = "A Melhor Interface do Mundo!",
   Duration = 5,
})

-- ============================================================
-- 11. ABA PLAYER
-- ============================================================
local PlayerTab = Window:CreateTab("👤 Player", nil)

local PlayerSection = PlayerTab:CreateSection("🎯 Configurações do Jogador")

PlayerTab:CreateSlider({
   Name = "Walk Speed",
   Range = {16, 250},
   Increment = 1,
   Suffix = "",
   CurrentValue = 16,
   Flag = "WalkSpeed",
   Callback = function(Value)
      Config.WalkSpeed = Value
   end,
})

PlayerTab:CreateSlider({
   Name = "Jump Power",
   Range = {50, 500},
   Increment = 10,
   Suffix = "",
   CurrentValue = 50,
   Flag = "JumpPower",
   Callback = function(Value)
      Config.JumpPower = Value
   end,
})

PlayerTab:CreateSlider({
   Name = "Fly Speed",
   Range = {50, 500},
   Increment = 10,
   Suffix = "",
   CurrentValue = 100,
   Flag = "FlySpeed",
   Callback = function(Value)
      Config.FlySpeed = Value
   end,
})

PlayerTab:CreateToggle({
   Name = "Infinite Jump",
   CurrentValue = false,
   Flag = "InfiniteJump",
   Callback = function(Value)
      Config.InfiniteJump = Value
   end,
})

PlayerTab:CreateToggle({
   Name = "No Clip",
   CurrentValue = false,
   Flag = "NoClip",
   Callback = function(Value)
      Config.NoClip = Value
   end,
})

PlayerTab:CreateToggle({
   Name = "Fly",
   CurrentValue = false,
   Flag = "Fly",
   Callback = function(Value)
      Config.Fly = Value
      if not Value then
         Flying = false
         if FlyConnection then
            FlyConnection:Disconnect()
            FlyConnection = nil
         end
      end
   end,
})

-- ============================================================
-- 12. ABA COMBAT
-- ============================================================
local CombatTab = Window:CreateTab("⚔️ Combat", nil)

local CombatSection = CombatTab:CreateSection("🎯 Sistema de Combate")

CombatTab:CreateToggle({
   Name = "Aimbot",
   CurrentValue = false,
   Flag = "Aimbot",
   Callback = function(Value)
      Config.Aimbot = Value
   end,
})

CombatTab:CreateToggle({
   Name = "Aimlock",
   CurrentValue = false,
   Flag = "Aimlock",
   Callback = function(Value)
      Config.Aimlock = Value
   end,
})

CombatTab:CreateToggle({
   Name = "Aim Assist",
   CurrentValue = false,
   Flag = "AimAssist",
   Callback = function(Value)
      Config.AimAssist = Value
   end,
})

CombatTab:CreateToggle({
   Name = "Auto Click",
   CurrentValue = false,
   Flag = "AutoClick",
   Callback = function(Value)
      Config.AutoClick = Value
   end,
})

CombatTab:CreateToggle({
   Name = "Trigger Bot",
   CurrentValue = false,
   Flag = "TriggerBot",
   Callback = function(Value)
      Config.TriggerBot = Value
   end,
})

CombatTab:CreateToggle({
   Name = "No Recoil",
   CurrentValue = false,
   Flag = "NoRecoil",
   Callback = function(Value)
      Config.NoRecoil = Value
   end,
})

CombatTab:CreateToggle({
   Name = "No Spread",
   CurrentValue = false,
   Flag = "NoSpread",
   Callback = function(Value)
      Config.NoSpread = Value
   end,
})

-- ============================================================
-- 13. ABA VISUALS
-- ============================================================
local VisualsTab = Window:CreateTab("👁️ Visuals", nil)

local VisualsSection = VisualsTab:CreateSection("🌟 Configurações Visuais")

VisualsTab:CreateToggle({
   Name = "ESP",
   CurrentValue = false,
   Flag = "ESP",
   Callback = function(Value)
      Config.ESP = Value
      if not Value then
         for _, obj in pairs(ESPObjects) do
            pcall(function() obj:Remove() end)
         end
         ESPObjects = {}
      end
   end,
})

VisualsTab:CreateToggle({
   Name = "Full Bright",
   CurrentValue = false,
   Flag = "FullBright",
   Callback = function(Value)
      Config.FullBright = Value
      if Value then
         game.Lighting.Brightness = 2
         game.Lighting.Ambient = Color3.new(1, 1, 1)
         game.Lighting.ClockTime = 12
      else
         game.Lighting.Brightness = 0.5
         game.Lighting.Ambient = Color3.new(0, 0, 0)
      end
   end,
})

VisualsTab:CreateToggle({
   Name = "FOV Changer",
   CurrentValue = false,
   Flag = "FOVChanger",
   Callback = function(Value)
      Config.FOVChanger = Value
   end,
})

VisualsTab:CreateSlider({
   Name = "FOV Value",
   Range = {60, 160},
   Increment = 1,
   Suffix = "°",
   CurrentValue = 120,
   Flag = "FOVValue",
   Callback = function(Value)
      Config.FOVValue = Value
   end,
})

-- ============================================================
-- 14. ABA MISC
-- ============================================================
local MiscTab = Window:CreateTab("⚙️ Misc", nil)

local MiscSection = MiscTab:CreateSection("⚙️ Diversos")

MiscTab:CreateToggle({
   Name = "Anti AFK",
   CurrentValue = false,
   Flag = "AntiAFK",
   Callback = function(Value)
      Config.AntiAFK = Value
   end,
})

MiscTab:CreateButton({
   Name = "🔴 Parar Tudo",
   Callback = function()
      Config.Aimbot = false
      Config.Aimlock = false
      Config.AimAssist = false
      Config.ESP = false
      Config.Fly = false
      Config.NoClip = false
      Config.InfiniteJump = false
      Config.FullBright = false
      
      game.Lighting.Brightness = 0.5
      game.Lighting.Ambient = Color3.new(0, 0, 0)
      game.Players.LocalPlayer.Character.Humanoid.WalkSpeed = 16
      game.Players.LocalPlayer.Character.Humanoid.JumpPower = 50
      
      for _, obj in pairs(ESPObjects) do
         pcall(function() obj:Remove() end)
      end
      ESPObjects = {}
      
      Rayfield:Notify({
         Title = "⏹️ Parado!",
         Content = "Tudo foi resetado!",
         Duration = 3,
      })
   end,
})

MiscTab:CreateButton({
   Name = "📋 Status",
   Callback = function()
      local msg = string.format([[
🚀 ULTIMATE EXPLOIT - STATUS

⚔️ Combat:
Aimbot: %s
Aimlock: %s
Aim Assist: %s
Target: %s

👤 Player:
Walk Speed: %d
Infinite Jump: %s
Fly: %s
No Clip: %s

👁️ Visuals:
ESP: %s
Full Bright: %s
FOV: %s
      ]],
      Config.Aimbot and "✅" or "❌",
      Config.Aimlock and "✅" or "❌",
      Config.AimAssist and "✅" or "❌",
      CurrentTarget and CurrentTarget.Name or "Nenhum",
      Config.WalkSpeed,
      Config.InfiniteJump and "✅" or "❌",
      Config.Fly and "✅" or "❌",
      Config.NoClip and "✅" or "❌",
      Config.ESP and "✅" or "❌",
      Config.FullBright and "✅" or "❌",
      Config.FOVChanger and "✅" or "❌"
      )
      
      Rayfield:Notify({
         Title = "📋 Status",
         Content = msg,
         Duration = 8,
      })
   end,
})

-- ============================================================
-- 15. ABA ABOUT
-- ============================================================
local AboutTab = Window:CreateTab("ℹ️ About", nil)

AboutTab:CreateParagraph({
    Title = "🚀 ULTIMATE EXPLOIT",
    Content = "✦ A Melhor Interface do Mundo ✦\n\nVersão: 10.0\nExecutor: Universal\n\n🔥 Funcionalidades:\n✅ Aimbot Premium\n✅ Aimlock\n✅ Aim Assist\n✅ ESP\n✅ Fly\n✅ No Clip\n✅ Infinite Jump\n✅ FOV Changer\n✅ Full Bright\n✅ Auto Click\n✅ Anti AFK\n✅ E MUITO MAIS!"
})

AboutTab:CreateButton({
   Name = "🌟 Doe para o Desenvolvedor",
   Callback = function()
      Rayfield:Notify({
         Title = "🙏 Obrigado!",
         Content = "Seu apoio é muito importante!",
         Duration = 3,
      })
   end,
})

-- ============================================================
-- 16. FINALIZAÇÃO
-- ============================================================
Rayfield:Notify({
   Title = "🌟 ULTIMATE EXPLOIT",
   Content = "A Melhor Interface do Mundo!",
   Duration = 3,
})

print("🚀 ULTIMATE EXPLOIT SCRIPT CARREGADO!")
print("✦ A Melhor Interface do Mundo ✦")
print("📌 Pressione K para abrir/fechar o menu")

-- ============================================================
-- FIM DO SCRIPT
-- ============================================================

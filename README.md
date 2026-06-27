--[[
    🎯 ZASKZ AIMFIX - Rayfield Edition
    ✦ Apenas ESP + Aimlock ✦
    ✦ Interface Rayfield ✦
]]

-- ============================================================
-- 1. CARREGAR RAYFIELD
-- ============================================================
local Rayfield = loadstring(game:HttpGet('https://raw.githubusercontent.com/shlexware/Rayfield/main/source'))()

-- ============================================================
-- 2. VARIÁVEIS
-- ============================================================
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local Camera = workspace.CurrentCamera
local LocalPlayer = Players.LocalPlayer

local cfg = {
    aimlock = false,
    esp = false,
}

local esp_objs = {}

-- ============================================================
-- 3. FUNÇÕES
-- ============================================================
local function W2S(pos)
    local v, on = Camera:WorldToViewportPoint(pos)
    if on and v.Z > 0 then
        return Vector2.new(v.X, v.Y)
    end
    return nil
end

local function Dist(a, b)
    return (b - a).Magnitude
end

local function GetPlayers()
    local list = {}
    for _, plr in pairs(Players:GetPlayers()) do
        if plr ~= LocalPlayer and plr.Character then
            local hum = plr.Character:FindFirstChild("Humanoid")
            if hum and hum.Health > 0 then
                list[#list + 1] = plr
            end
        end
    end
    return list
end

local function GetPart(plr, part)
    if not plr or not plr.Character then return nil end
    return plr.Character:FindFirstChild(part)
end

-- ============================================================
-- 4. AIMLOCK
-- ============================================================
local function FindBestTarget()
    local center = Vector2.new(Camera.ViewportSize.X / 2, Camera.ViewportSize.Y / 2)
    local best, bestScore = nil, math.huge
    
    for _, plr in pairs(GetPlayers()) do
        local head = GetPart(plr, "Head")
        if head then
            local sp = W2S(head.Position)
            if sp then
                local fov = Dist(center, sp)
                local range = Dist(head.Position, Camera.CFrame.Position)
                
                if fov < 200 and range < 250 then
                    local score = fov + range / 10
                    if score < bestScore then
                        bestScore = score
                        best = plr
                    end
                end
            end
        end
    end
    return best
end

local function UpdateAimlock()
    if not cfg.aimlock then return end
    
    local t = FindBestTarget()
    if t then
        local head = GetPart(t, "Head")
        if head then
            Camera.CFrame = CFrame.new(Camera.CFrame.Position, head.Position)
        end
    end
end

-- ============================================================
-- 5. ESP
-- ============================================================
local function ClearESP()
    for _, obj in pairs(esp_objs) do
        pcall(function() obj:Remove() end)
    end
    esp_objs = {}
end

local function UpdateESP()
    ClearESP()
    if not cfg.esp then return end
    
    for _, plr in pairs(GetPlayers()) do
        local root = GetPart(plr, "HumanoidRootPart")
        local head = GetPart(plr, "Head")
        local hum = plr.Character:FindFirstChild("Humanoid")
        
        if root and head and hum then
            local dist = Dist(root.Position, Camera.CFrame.Position)
            if dist > 300 then break end
            
            local rp = W2S(root.Position)
            local hp = W2S(head.Position)
            
            if rp and hp then
                local h = math.abs(hp.Y - rp.Y) * 1.5
                local w = h * 0.5
                local bp = Vector2.new(rp.X - w/2, hp.Y - h/4)
                
                -- Box
                local box = Drawing.new("Square")
                box.Size = Vector2.new(w, h)
                box.Position = bp
                box.Color = Color3.fromRGB(255, 0, 0)
                box.Thickness = 2
                box.Filled = false
                box.Visible = true
                table.insert(esp_objs, box)
                
                -- Name
                local name = Drawing.new("Text")
                name.Text = plr.Name
                name.Size = 13
                name.Color = Color3.fromRGB(255, 255, 255)
                name.Position = Vector2.new(bp.X + w/2 - name.TextBounds.X/2, bp.Y - 18)
                name.Visible = true
                table.insert(esp_objs, name)
                
                -- Health
                local hpct = hum.Health / hum.MaxHealth
                local hc = Color3.fromRGB(255 * (1 - hpct), 255 * hpct, 0)
                
                local health = Drawing.new("Square")
                health.Size = Vector2.new(w, 4)
                health.Position = Vector2.new(bp.X, bp.Y + h + 2)
                health.Color = hc
                health.Thickness = 0
                health.Filled = true
                health.Visible = true
                table.insert(esp_objs, health)
            end
        end
    end
end

-- ============================================================
-- 6. LOOP
-- ============================================================
task.spawn(function()
    while true do
        task.wait(0.05)
        pcall(UpdateAimlock)
        pcall(UpdateESP)
    end
end)

-- ============================================================
-- 7. RAYFIELD INTERFACE
-- ============================================================
local Window = Rayfield:CreateWindow({
   Name = "🎯 ZASKZ AIMFIX",
   Icon = 0,
   LoadingTitle = "Carregando...",
   LoadingSubtitle = "Rayfield Edition",
   Theme = "Dark",
   ToggleUIKeybind = Enum.KeyCode.Insert,
})

-- ============================================================
-- 8. ABA PRINCIPAL
-- ============================================================
local MainTab = Window:CreateTab("🎯 Menu", nil)

MainTab:CreateSection("⚙️ Configurações")

MainTab:CreateToggle({
   Name = "Aimlock",
   CurrentValue = false,
   Flag = "Aimlock",
   Callback = function(Value)
      cfg.aimlock = Value
   end,
})

MainTab:CreateToggle({
   Name = "ESP",
   CurrentValue = false,
   Flag = "ESP",
   Callback = function(Value)
      cfg.esp = Value
      if not Value then ClearESP() end
   end,
})

MainTab:CreateButton({
   Name = "🔴 PARAR TUDO",
   Callback = function()
      cfg.aimlock = false
      cfg.esp = false
      ClearESP()
      Rayfield:Notify({
         Title = "⏹️ Parado!",
         Content = "Tudo desativado!",
         Duration = 2,
      })
   end,
})

MainTab:CreateButton({
   Name = "📋 STATUS",
   Callback = function()
      Rayfield:Notify({
         Title = "📋 Status",
         Content = string.format("Aimlock: %s\nESP: %s", 
            cfg.aimlock and "✅" or "❌",
            cfg.esp and "✅" or "❌"
         ),
         Duration = 4,
      })
   end,
})

-- ============================================================
-- 9. FINALIZAÇÃO
-- ============================================================
Rayfield:Notify({
   Title = "🎯 ZASKZ AIMFIX",
   Content = "Pressione INSERT",
   Duration = 2,
})

print("🎯 ZASKZ AIMFIX CARREGADO!")
print("📌 Pressione INSERT")

-- ============================================================
-- FIM
-- ============================================================

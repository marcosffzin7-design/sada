--[[
    🎯 ZASKZ AIMFIX - Ultra Simples
    ✦ Apenas Aimlock + ESP ✦
]]

-- 1. CARREGAR RAYFIELD
local Rayfield = loadstring(game:HttpGet('https://raw.githubusercontent.com/shlexware/Rayfield/main/source'))()

-- 2. VARIÁVEIS
local p = game:GetService("Players").LocalPlayer
local c = workspace.CurrentCamera
local esp_objs = {}
local cfg = {aimlock=false, esp=false}

-- 3. FUNÇÕES
local function W2S(pos)
    local v, on = c:WorldToViewportPoint(pos)
    return on and v.Z > 0 and Vector2.new(v.X, v.Y) or nil
end

local function Dist(a, b) return (b - a).Magnitude end

local function GetPlayers()
    local list = {}
    for _, plr in pairs(game:GetService("Players"):GetPlayers()) do
        if plr ~= p and plr.Character then
            local hum = plr.Character:FindFirstChild("Humanoid")
            if hum and hum.Health > 0 then list[#list+1] = plr end
        end
    end
    return list
end

local function GetHead(plr)
    return plr and plr.Character and plr.Character:FindFirstChild("Head") or nil
end

-- 4. AIMLOCK
local function FindTarget()
    local center = Vector2.new(c.ViewportSize.X/2, c.ViewportSize.Y/2)
    local best, bestScore = nil, math.huge
    for _, plr in pairs(GetPlayers()) do
        local head = GetHead(plr)
        if head then
            local sp = W2S(head.Position)
            if sp then
                local fov = Dist(center, sp)
                local range = Dist(head.Position, c.CFrame.Position)
                if fov < 200 and range < 250 then
                    local score = fov + range/10
                    if score < bestScore then bestScore = score best = plr end
                end
            end
        end
    end
    return best
end

-- 5. ESP
local function ClearESP()
    for _, obj in pairs(esp_objs) do pcall(function() obj:Remove() end) end
    esp_objs = {}
end

-- 6. LOOP
task.spawn(function()
    while true do
        task.wait(0.05)
        
        -- Aimlock
        if cfg.aimlock then
            local t = FindTarget()
            if t then
                local head = GetHead(t)
                if head then c.CFrame = CFrame.new(c.CFrame.Position, head.Position) end
            end
        end
        
        -- ESP
        ClearESP()
        if cfg.esp then
            for _, plr in pairs(GetPlayers()) do
                local root = plr.Character:FindFirstChild("HumanoidRootPart")
                local head = GetHead(plr)
                local hum = plr.Character:FindFirstChild("Humanoid")
                if root and head and hum then
                    local dist = Dist(root.Position, c.CFrame.Position)
                    if dist < 300 then
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
                            box.Color = Color3.fromRGB(255,0,0)
                            box.Thickness = 2
                            box.Filled = false
                            box.Visible = true
                            table.insert(esp_objs, box)
                            
                            -- Name
                            local name = Drawing.new("Text")
                            name.Text = plr.Name
                            name.Size = 13
                            name.Color = Color3.fromRGB(255,255,255)
                            name.Position = Vector2.new(bp.X + w/2 - name.TextBounds.X/2, bp.Y - 18)
                            name.Visible = true
                            table.insert(esp_objs, name)
                            
                            -- Health
                            local hpct = hum.Health / hum.MaxHealth
                            local health = Drawing.new("Square")
                            health.Size = Vector2.new(w, 4)
                            health.Position = Vector2.new(bp.X, bp.Y + h + 2)
                            health.Color = Color3.fromRGB(255*(1-hpct), 255*hpct, 0)
                            health.Thickness = 0
                            health.Filled = true
                            health.Visible = true
                            table.insert(esp_objs, health)
                        end
                    end
                end
            end
        end
    end
end)

-- 7. RAYFIELD
local Window = Rayfield:CreateWindow({
   Name = "🎯 ZASKZ AIMFIX",
   Icon = 0,
   LoadingTitle = "Carregando...",
   Theme = "Dark",
   ToggleUIKeybind = Enum.KeyCode.Insert,
})

local MainTab = Window:CreateTab("🎯 Menu", nil)

MainTab:CreateSection("⚙️ Config")

MainTab:CreateToggle({
   Name = "Aimlock",
   CurrentValue = false,
   Flag = "Aimlock",
   Callback = function(v) cfg.aimlock = v end,
})

MainTab:CreateToggle({
   Name = "ESP",
   CurrentValue = false,
   Flag = "ESP",
   Callback = function(v) cfg.esp = v if not v then ClearESP() end end,
})

MainTab:CreateButton({
   Name = "🔴 PARAR",
   Callback = function()
      cfg.aimlock = false
      cfg.esp = false
      ClearESP()
      Rayfield:Notify({Title="⏹️ Parado!", Content="Tudo desativado!", Duration=2})
   end,
})

-- 8. FIM
Rayfield:Notify({Title="🎯 CARREGADO!", Content="Pressione INSERT", Duration=2})
print("🎯 ZASKZ AIMFIX - Ultra Simples")

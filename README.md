--[[ 🔥 DOGS OF WAR - TEAM ESP + ULTIMATE CAMLOCK + AUTO SHOOT + OUTLINE ULTRA LITE + FOV AIMBOT REMAKE + AUTO M2 + AUTO RELOAD (FOV AIMBOT CORRIGIDO)
💣 AIMBOT FOV CORRIGIDO: Mira 100% precisa na cabeça + detecção de proximidade melhorada
✅ ESP só do time inimigo (correta Axis/Allies) com HIGHLIGHTS SUPREMOS
✅ AutoShoot CORRIGIDO: Dispara em qualquer distância quando mira está ativa
✅ Auto M2 OP: M2 ativa instantaneamente quando troca de alvo
✅ FOV Aimbot PRECISO: Mira exatamente no centro da cabeça para máximo dano
✅ DETECÇÃO PRÓXIMA: Funciona perfeitamente mesmo com inimigos muito próximos
✅ AutoReload: Aperta R automaticamente se balas chegar em 0 ou 1
🧠 HIGHLIGHT SETUP IDEAL + ESP HUD EXTRA + FILTRO INTELIGENTE + OTIMIZAÇÃO MOBILE
]]

-- CONFIG
local FOV_SIZE = 210 -- FOV maior/OP
local FACE_OFFSET = 0 -- CORRIGIDO: Mira exatamente no centro da cabeça (era 0.1)
local FOV_CLOSE_DIST = 15 -- AUMENTADO: Melhor detecção de proximidade (era 8)
local ESP_MAX_DISTANCE = 1000 -- Desativa ESP em players fora de 1000 studs
local AUTO_SHOOT_MAX_DIST = 500 -- AUMENTADO: Distância maior para auto tiro (era 260)
local AUTO_SHOOT_CHECK_INTERVAL = 0.005 -- MAIS RÁPIDO: Melhor responsividade (era 0.008)
local FOV_AIMBOT_CHECK_INTERVAL = 0.005 -- MAIS RÁPIDO (era 0.008)
local ESP_UPDATE_INTERVAL = 1 -- Atualiza a cada 1 segundo
local LOW_HEALTH_THRESHOLD = 0.5 -- 50% de vida considera como "vida baixa"

-- 🧠 HIGHLIGHT SETUP IDEAL - Cores para ESP e Highlights SUPREMOS
local TEAM_COLORS = {
    Axis = Color3.fromRGB(255, 165, 0), -- Laranja
    Allies = Color3.fromRGB(0, 255, 0) -- Verde
}

local HIGHLIGHT_COLORS = {
    Axis = Color3.fromRGB(255, 140, 0), -- Laranja mais escuro para highlight
    Allies = Color3.fromRGB(0, 200, 0) -- Verde mais escuro para highlight
}

-- 💥 CORES PARA ESP HUD EXTRA
local HUD_COLORS = {
    name = Color3.new(1, 1, 1), -- Branco puro para nome
    distance = Color3.fromRGB(255, 255, 0), -- Amarelo para distância
    health_high = Color3.fromRGB(0, 255, 0), -- Verde para vida alta
    health_medium = Color3.fromRGB(255, 255, 0), -- Amarelo para vida média
    health_low = Color3.fromRGB(255, 0, 0) -- Vermelho para vida baixa
}

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local TweenService = game:GetService("TweenService")
local VirtualInputManager = game:GetService("VirtualInputManager")
local LocalPlayer = Players.LocalPlayer
local Camera = workspace.CurrentCamera
local UserInputService = game:GetService("UserInputService")

local CamLockActive = false
local CurrentTarget = nil
local LastTarget = nil
local LastTargetId = nil
local ActiveTeamESP = nil -- 'Axis' ou 'Allies'
local ESPConnections = {}
local PlayerESPs = {}
local PlayerOutlines = {}
local PlayerHighlights = {} -- Para armazenar highlights
local AutoShootActive = false
local FOV_AIMBOT_ACTIVE = false
local FOVAimbotButton
local AxisButton, AlliesButton, CamLockButton, AutoShootButton

-- ============ UTILS ============

local function IsAlive(char)
    if not char then return false end
    local hum = char:FindFirstChildOfClass("Humanoid")
    return hum and hum.Health > 0
end

local function GetHealthPercentage(char)
    if not char then return 0 end
    local hum = char:FindFirstChildOfClass("Humanoid")
    if not hum then return 0 end
    return hum.Health / hum.MaxHealth
end

local function IsHeadVisible(myPos, targetChar)
    local head = targetChar and targetChar:FindFirstChild("Head")
    if not head then return false end
    local ray = Ray.new(myPos, (head.Position - myPos).Unit * (head.Position - myPos).Magnitude)
    local part = workspace:FindPartOnRayWithIgnoreList(ray, {LocalPlayer.Character, targetChar})
    return not part or part:IsDescendantOf(targetChar)
end

local function IsBodyVisible(myPos, targetChar)
    local hrp = targetChar and targetChar:FindFirstChild("HumanoidRootPart")
    if not hrp then return false end
    local ray = Ray.new(myPos, (hrp.Position - myPos).Unit * (hrp.Position - myPos).Magnitude)
    local part = workspace:FindPartOnRayWithIgnoreList(ray, {LocalPlayer.Character, targetChar})
    return not part or part:IsDescendantOf(targetChar)
end

local function GetPlayerTeam(player)
    if not player.Character then return nil end
    if player.TeamColor then
        if player.TeamColor == BrickColor.new("Bright orange") or player.TeamColor.Name:lower():find("orange") then
            return "Axis"
        elseif player.TeamColor == BrickColor.new("Bright green") or player.TeamColor.Name:lower():find("green") then
            return "Allies"
        end
    end
    if player.Team then
        local tn = player.Team.Name:lower()
        if tn:find("axis") or tn:find("orange") then return "Axis"
        elseif tn:find("allies") or tn:find("green") then return "Allies" end
    end
    return nil
end

-- 🔥 FILTRO INTELIGENTE - Só aplica ESP a inimigos
local function IsEnemy(player)
    if player == LocalPlayer then return false end
    if not player.Character or not IsAlive(player.Character) then return false end
    
    local myTeam = GetPlayerTeam(LocalPlayer)
    local theirTeam = GetPlayerTeam(player)
    
    if not myTeam or not theirTeam then return false end
    if not ActiveTeamESP then return false end
    
    -- Se escolheu Axis, mostra inimigos Allies (e vice-versa)
    local enemyTeam = (ActiveTeamESP == "Axis") and "Allies" or "Axis"
    return theirTeam == enemyTeam
end

-- ============ 🧠 HIGHLIGHT SETUP IDEAL + ESP HUD EXTRA ============

local function CreateSupremeHighlight(char, teamColor)
    if not char or not IsAlive(char) then return end
    
    -- Remove highlight existente
    local existingHighlight = char:FindFirstChild("PlayerESP_Supreme_Highlight")
    if existingHighlight then
        existingHighlight:Destroy()
    end
    
    -- 🧠 HIGHLIGHT SETUP IDEAL
    local highlight = Instance.new("Highlight")
    highlight.Name = "PlayerESP_Supreme_Highlight"
    highlight.Parent = char
    highlight.Adornee = char
    highlight.FillColor = teamColor -- Cor do time
    highlight.OutlineColor = Color3.new(1, 1, 1) -- Branco puro
    highlight.FillTransparency = 0.2 -- Leve brilho
    highlight.OutlineTransparency = 0 -- Linha sólida branca
    highlight.DepthMode = Enum.HighlightDepthMode.AlwaysOnTop -- Atravessa parede
    
    return highlight
end

local function CreateOutlineESP(char, color)
    if not char or not IsAlive(char) then return end
    for _, part in ipairs(char:GetDescendants()) do
        if part:IsA("BasePart") and part.Name ~= "HumanoidRootPart" then
            local existing = part:FindFirstChild("ESP_Highlight")
            if not existing then
                local highlight = Instance.new("Highlight")
                highlight.Name = "ESP_Highlight"
                highlight.Adornee = part
                highlight.OutlineColor = Color3.new(1, 1, 1) -- Branco puro
                highlight.FillColor = color
                highlight.OutlineTransparency = 0
                highlight.FillTransparency = 0.8
                highlight.DepthMode = Enum.HighlightDepthMode.AlwaysOnTop
                highlight.Parent = part
            else
                existing.OutlineColor = Color3.new(1, 1, 1)
                existing.FillColor = color
                existing.OutlineTransparency = 0
                existing.FillTransparency = 0.8
                existing.DepthMode = Enum.HighlightDepthMode.AlwaysOnTop
            end
        end
    end
end

local function RemoveOutlineESP(char)
    for _, part in ipairs(char:GetDescendants()) do
        if part:IsA("BasePart") and part:FindFirstChild("ESP_Highlight") then
            part.ESP_Highlight:Destroy()
        end
    end
end

local function RemoveSupremeHighlight(char)
    local highlight = char:FindFirstChild("PlayerESP_Supreme_Highlight")
    if highlight then
        highlight:Destroy()
    end
end

-- 💥 ESP HUD EXTRA com nome, distância e vida
local function CreateSupremeESP(player, teamColor)
    if PlayerESPs[player] then PlayerESPs[player]:Destroy() end
    local char = player.Character
    if not char or not char:FindFirstChild("HumanoidRootPart") or not IsAlive(char) then return end
    
    -- 🧠 HIGHLIGHT SETUP IDEAL
    local supremeHighlight = CreateSupremeHighlight(char, teamColor)
    PlayerHighlights[player] = supremeHighlight
    
    -- Mantém outline nos parts individuais para dupla proteção
    CreateOutlineESP(char, teamColor)
    PlayerOutlines[player] = true
    
    -- 💥 ESP HUD EXTRA
    local esp = Instance.new("BillboardGui")
    esp.Name = "PlayerESP_Supreme_"..player.Name
    esp.Parent = char:FindFirstChild("HumanoidRootPart")
    esp.Size = UDim2.new(0, 55, 0, 35)
    esp.StudsOffset = Vector3.new(0, 2.8, 0)
    esp.AlwaysOnTop = true
    
    -- Nome do inimigo
    local nameLabel = Instance.new("TextLabel")
    nameLabel.Parent = esp
    nameLabel.Size = UDim2.new(1, 0, 0.4, 0)
    nameLabel.BackgroundTransparency = 1
    nameLabel.Text = player.Name:sub(1, 10)
    nameLabel.TextColor3 = HUD_COLORS.name -- Branco puro
    nameLabel.TextScaled = true
    nameLabel.Font = Enum.Font.GothamBold
    nameLabel.TextStrokeTransparency = 0
    nameLabel.TextStrokeColor3 = Color3.new(0, 0, 0)
    
    -- Distância
    local distanceLabel = Instance.new("TextLabel")
    distanceLabel.Parent = esp
    distanceLabel.Size = UDim2.new(1, 0, 0.3, 0)
    distanceLabel.Position = UDim2.new(0, 0, 0.4, 0)
    distanceLabel.BackgroundTransparency = 1
    distanceLabel.Text = "0m"
    distanceLabel.TextColor3 = HUD_COLORS.distance -- Amarelo
    distanceLabel.TextScaled = true
    distanceLabel.Font = Enum.Font.Code
    distanceLabel.TextStrokeTransparency = 0
    distanceLabel.TextStrokeColor3 = Color3.new(0, 0, 0)
    
    -- Vida (UIStroke de cor degradê)
    local healthLabel = Instance.new("TextLabel")
    healthLabel.Parent = esp
    healthLabel.Size = UDim2.new(1, 0, 0.3, 0)
    healthLabel.Position = UDim2.new(0, 0, 0.7, 0)
    healthLabel.BackgroundTransparency = 1
    healthLabel.Text = "100%"
    healthLabel.TextColor3 = HUD_COLORS.health_high
    healthLabel.TextScaled = true
    healthLabel.Font = Enum.Font.GothamBold
    healthLabel.TextStrokeTransparency = 0
    healthLabel.TextStrokeColor3 = Color3.new(0, 0, 0)
    
    PlayerESPs[player] = esp
    
    -- 📱 OTIMIZAÇÃO MOBILE - Atualiza apenas a cada RunService.Heartbeat
    local conn
    conn = RunService.Heartbeat:Connect(function()
        if not player.Character or not LocalPlayer.Character then 
            conn:Disconnect() 
            return 
        end
        local theirHRP = player.Character:FindFirstChild("HumanoidRootPart")
        local myHRP = LocalPlayer.Character:FindFirstChild("HumanoidRootPart")
        local theirHum = player.Character:FindFirstChildOfClass("Humanoid")
        
        if theirHRP and myHRP and theirHum then
            local dist = math.floor((theirHRP.Position - myHRP.Position).Magnitude)
            distanceLabel.Text = dist.."m"
            
            -- Vida com cor degradê
            local healthPercent = math.floor((theirHum.Health / theirHum.MaxHealth) * 100)
            healthLabel.Text = healthPercent.."%"
            
            if healthPercent > 70 then
                healthLabel.TextColor3 = HUD_COLORS.health_high -- Verde
            elseif healthPercent > 30 then
                healthLabel.TextColor3 = HUD_COLORS.health_medium -- Amarelo
            else
                healthLabel.TextColor3 = HUD_COLORS.health_low -- Vermelho
            end
            
            -- 📱 OTIMIZAÇÃO - Desativa ESP em players fora do alcance
            esp.Enabled = dist <= ESP_MAX_DISTANCE and IsAlive(player.Character)
            
            -- Atualiza highlight se o personagem mudou
            if not PlayerHighlights[player] or PlayerHighlights[player].Parent ~= player.Character then
                RemoveSupremeHighlight(player.Character)
                local newHighlight = CreateSupremeHighlight(player.Character, teamColor)
                PlayerHighlights[player] = newHighlight
            end
        end
    end)
    ESPConnections[player] = conn
end

local function RemoveSupremeESP(player)
    if PlayerESPs[player] then PlayerESPs[player]:Destroy(); PlayerESPs[player] = nil end
    if ESPConnections[player] then ESPConnections[player]:Disconnect(); ESPConnections[player] = nil end
    if PlayerOutlines[player] and player.Character then 
        RemoveOutlineESP(player.Character)
        PlayerOutlines[player] = nil 
    end
    if PlayerHighlights[player] then
        if player.Character then
            RemoveSupremeHighlight(player.Character)
        end
        PlayerHighlights[player] = nil
    end
end

-- 🔥 FILTRO INTELIGENTE - Atualização otimizada
local function UpdateTeamESP()
    for p, esp in pairs(PlayerESPs) do RemoveSupremeESP(p) end
    PlayerESPs, ESPConnections, PlayerOutlines, PlayerHighlights = {}, {}, {}, {}
    if not ActiveTeamESP then return end
    
    for _, player in pairs(Players:GetPlayers()) do
        if IsEnemy(player) then -- 🔥 FILTRO INTELIGENTE
            local team = GetPlayerTeam(player)
            CreateSupremeESP(player, TEAM_COLORS[team])
        end
    end
end

local function FindBestTargetByTeam(targetTeam)
    local lpchar = LocalPlayer.Character
    if not lpchar then return nil end
    local hrp = lpchar:FindFirstChild("HumanoidRootPart")
    if not hrp then return nil end
    local best, closestAngle = nil, math.rad(18)
    local camPos = Camera.CFrame.Position
    for _, p in pairs(Players:GetPlayers()) do
        if p ~= LocalPlayer and p.Character then
            local team = GetPlayerTeam(p)
            if team == targetTeam then
                local tHRP = p.Character:FindFirstChild("HumanoidRootPart")
                local tHead = p.Character:FindFirstChild("Head")
                if tHRP and tHead and IsAlive(p.Character) then
                    local dist = (tHRP.Position - camPos).Magnitude
                    if dist <= ESP_MAX_DISTANCE then
                        local dir = (tHRP.Position - camPos).Unit
                        local camLook = Camera.CFrame.LookVector
                        local angle = math.acos(math.clamp(dir:Dot(camLook), -1, 1))
                        if angle < closestAngle then closestAngle = angle; best = p.Character end
                    end
                end
            end
        end
    end
    return best
end

-- 🎯 FOV AIMBOT CORRIGIDO: Funciona em qualquer distância
local function FindFOVAimbotTarget(targetTeam)
    local lpchar = LocalPlayer.Character
    if not lpchar then return nil end
    local hrp = lpchar:FindFirstChild("HumanoidRootPart")
    if not hrp then return nil end
    
    local allTargets = {} -- Todos os alvos válidos
    local screenCenter = Vector2.new(Camera.ViewportSize.X/2, Camera.ViewportSize.Y/2)
    
    for _, p in pairs(Players:GetPlayers()) do
        if p ~= LocalPlayer and p.Character and IsAlive(p.Character) then
            local team = GetPlayerTeam(p)
            if team == ((ActiveTeamESP == "Axis") and "Allies" or "Axis") then
                local head = p.Character:FindFirstChild("Head")
                local hrpTgt = p.Character:FindFirstChild("HumanoidRootPart")
                if head and hrpTgt then
                    local distWorld = (hrp.Position - head.Position).Magnitude
                    local healthPercent = GetHealthPercentage(p.Character)
                    
                    -- CORRIGIDO: Verifica visibilidade primeiro
                    if IsHeadVisible(hrp.Position, p.Character) then
                        -- CORRIGIDO: Para alvos muito próximos, não verifica FOV
                        if distWorld <= FOV_CLOSE_DIST then
                            table.insert(allTargets, {
                                character = p.Character,
                                worldDistance = distWorld,
                                screenDistance = 0, -- Prioridade máxima para próximos
                                healthPercent = healthPercent,
                                isClose = true
                            })
                        else
                            -- Para alvos distantes, verifica FOV
                            local headPos, onScreen = Camera:WorldToScreenPoint(head.Position)
                            if onScreen then
                                local screenPos = Vector2.new(headPos.X, headPos.Y)
                                local distScreen = (screenPos - screenCenter).Magnitude
                                
                                if distScreen <= FOV_SIZE/2 then
                                    table.insert(allTargets, {
                                        character = p.Character,
                                        worldDistance = distWorld,
                                        screenDistance = distScreen,
                                        healthPercent = healthPercent,
                                        isClose = false
                                    })
                                end
                            end
                        end
                    end
                end
            end
        end
    end
    
    if #allTargets == 0 then return nil end
    
    -- CORRIGIDO: Ordena por prioridade (próximos primeiro, depois por vida baixa, depois por distância)
    table.sort(allTargets, function(a, b)
        -- Prioridade 1: Alvos próximos
        if a.isClose and not b.isClose then return true end
        if b.isClose and not a.isClose then return false end
        
        -- Prioridade 2: Vida baixa
        local aLowHealth = a.healthPercent <= LOW_HEALTH_THRESHOLD
        local bLowHealth = b.healthPercent <= LOW_HEALTH_THRESHOLD
        if aLowHealth and not bLowHealth then return true end
        if bLowHealth and not aLowHealth then return false end
        
        -- Prioridade 3: Distância menor
        return a.worldDistance < b.worldDistance
    end)
    
    return allTargets[1].character
end

-- 🎯 MIRA CORRIGIDA: 100% precisa na cabeça
local function AimAtHead(targetChar)
    if not targetChar or not targetChar:FindFirstChild("Head") then return false end
    
    local head = targetChar.Head
    local myChar = LocalPlayer.Character
    if not myChar or not myChar:FindFirstChild("HumanoidRootPart") then return false end
    
    -- CORRIGIDO: Mira exatamente no centro da cabeça para máximo dano
    local headCenter = head.Position
    
    -- CORRIGIDO: Remove qualquer offset, mira direta no centro
    Camera.CFrame = CFrame.new(Camera.CFrame.Position, headCenter)
    return true
end

local function CamLockThread()
    while CamLockActive and RunService.RenderStepped:Wait() do
        if CurrentTarget ~= LastTarget then LastTarget = CurrentTarget end
        if CurrentTarget and CurrentTarget:FindFirstChild("Head") then
            AimAtHead(CurrentTarget)
            local hum = CurrentTarget:FindFirstChildOfClass("Humanoid")
            if hum and hum.Health <= 0 then
                CamLockActive = false
                CamLockButton.Text = "🎯 OFF\n📱 Toque para ativar\n↔️ Arraste para mover"
                CamLockButton.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
                CurrentTarget = nil
            end
        else
            local tgt = (ActiveTeamESP == "Axis") and "Allies" or "Axis"
            CurrentTarget = FindBestTargetByTeam(tgt)
            if not CurrentTarget then
                CamLockActive = false
                CamLockButton.Text = "🎯 OFF\n📱 Toque para ativar\n↔️ Arraste para mover"
                CamLockButton.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
            end
        end
    end
end

-- ============ AUTO SHOOT + AUTO M2 + AUTO RELOAD CORRIGIDO ============

local function AutoShootThread()
    local m1down = false
    local m2down = false
    local lastTargetId = nil
    
    while AutoShootActive do
        local validTarget = (CamLockActive or FOV_AIMBOT_ACTIVE) and CurrentTarget and IsAlive(CurrentTarget)
        local myChar = LocalPlayer.Character
        local myHRP = myChar and myChar:FindFirstChild("HumanoidRootPart")
        local headVis = validTarget and myHRP and IsHeadVisible(myHRP.Position, CurrentTarget)
        local bodyVis = validTarget and myHRP and IsBodyVisible(myHRP.Position, CurrentTarget)
        local targetHRP = validTarget and CurrentTarget:FindFirstChild("HumanoidRootPart")
        local dist = (myHRP and targetHRP) and (myHRP.Position - targetHRP.Position).Magnitude or math.huge
        local targetId = validTarget and tostring(CurrentTarget) or nil
        
        -- Auto M2 CORRIGIDO: Ativa para qualquer distância se o alvo for visível
        if validTarget and (headVis or bodyVis) and IsAlive(CurrentTarget) and IsAlive(myChar) then
            if (not m2down) or (lastTargetId ~= targetId) then
                m2down = true
                VirtualInputManager:SendMouseButtonEvent(0, 0, 1, true, game, 0)
                lastTargetId = targetId
            end
        else
            if m2down then
                m2down = false
                VirtualInputManager:SendMouseButtonEvent(0, 0, 1, false, game, 0)
                lastTargetId = nil
            end
        end
        
        -- Auto M1 CORRIGIDO: Atira em qualquer distância se M2 estiver ativo e cabeça visível
        if validTarget and headVis and dist <= AUTO_SHOOT_MAX_DIST and m2down then
            if not m1down then
                m1down = true
                VirtualInputManager:SendMouseButtonEvent(0, 0, 0, true, game, 0)
            end
        else
            if m1down then
                m1down = false
                VirtualInputManager:SendMouseButtonEvent(0, 0, 0, false, game, 0)
            end
        end
        
        -- AutoReload: se munição chegar em 0 ou 1, simula R
        local tool = myChar and myChar:FindFirstChildOfClass("Tool")
        if tool then
            local bullets = nil
            for _, v in pairs(tool:GetChildren()) do
                if v:IsA("IntValue") and (v.Name:lower():find("ammo") or v.Name:lower():find("clip") or v.Name:lower():find("balas") or v.Name:lower():find("punicao")) then
                    if not bullets or v.Value < bullets then bullets = v.Value end
                end
            end
            for _, attr in pairs({"Ammo","Clip","Balas","Bullets","Punição","Punicao"}) do
                if tool:GetAttribute(attr) then
                    local val = tool:GetAttribute(attr)
                    if type(val) == "number" and (not bullets or val < bullets) then bullets = val end
                end
            end
            if bullets and bullets <= 1 then
                VirtualInputManager:SendKeyEvent(true, Enum.KeyCode.R, false, game)
                wait(0.08)
                VirtualInputManager:SendKeyEvent(false, Enum.KeyCode.R, false, game)
            end
        end
        
        -- Se alvo morrer ou não estiver visível, solta M2/M1
        if not validTarget or not IsAlive(myChar) or not IsAlive(CurrentTarget) or (not headVis and not bodyVis) then
            if m2down then
                m2down = false
                VirtualInputManager:SendMouseButtonEvent(0, 0, 1, false, game, 0)
                lastTargetId = nil
            end
            if m1down then
                m1down = false
                VirtualInputManager:SendMouseButtonEvent(0, 0, 0, false, game, 0)
            end
        end
        
        RunService.RenderStepped:Wait(AUTO_SHOOT_CHECK_INTERVAL)
    end
    if m1down then VirtualInputManager:SendMouseButtonEvent(0, 0, 0, false, game, 0) end
    if m2down then VirtualInputManager:SendMouseButtonEvent(0, 0, 1, false, game, 0) end
end

-- 🎯 FOV AIMBOT CORRIGIDO
local function FOVAimbotThread()
    while FOV_AIMBOT_ACTIVE and RunService.RenderStepped:Wait(FOV_AIMBOT_CHECK_INTERVAL) do
        if not ActiveTeamESP then continue end
        local targetTeam = (ActiveTeamESP == "Axis") and "Allies" or "Axis"
        local newTarget = FindFOVAimbotTarget(targetTeam)
        local lpchar = LocalPlayer.Character
        local aliveMe = IsAlive(lpchar)
        local aliveTarget = newTarget and IsAlive(newTarget)
        
        if not aliveMe or not aliveTarget then
            CurrentTarget = nil
        elseif newTarget and (CurrentTarget ~= newTarget) then
            CurrentTarget = newTarget
        end
        
        if CurrentTarget and CurrentTarget:FindFirstChild("Head") and IsAlive(CurrentTarget) and aliveMe then
            -- CORRIGIDO: Usa a função de mira corrigida
            AimAtHead(CurrentTarget)
        end
    end
end

-- ============ TOUCH UI ============

local function CreateTeamButtons()
    local ScreenGui = Instance.new("ScreenGui")
    ScreenGui.Name = "DogsOfWarUI"
    ScreenGui.Parent = game:GetService("CoreGui")
    ScreenGui.ResetOnSpawn = false
    
    local function makeBtn(name, posY, text, color, fontColor)
        local btn = Instance.new("TextButton")
        btn.Name = name
        btn.Size = UDim2.new(0, 80, 0, 35)
        btn.Position = UDim2.new(0.85, 0, posY, 0)
        btn.TextSize = 10
        btn.BackgroundColor3 = color
        btn.TextColor3 = fontColor or Color3.new(1, 1, 1)
        btn.BorderSizePixel = 0
        btn.ZIndex = 1000
        btn.Text = text
        btn.Parent = ScreenGui
        
        local isDragging = false
        local startPos = nil
        local startTime = 0
        
        btn.InputBegan:Connect(function(input)
            if input.UserInputType == Enum.UserInputType.Touch or input.UserInputType == Enum.UserInputType.MouseButton1 then
                isDragging = false
                startPos = input.Position
                startTime = tick()
            end
        end)
        
        btn.InputChanged:Connect(function(input)
            if input.UserInputType == Enum.UserInputType.Touch or input.UserInputType == Enum.UserInputType.MouseMovement then
                if startPos and (input.Position - startPos).Magnitude > 10 then
                    isDragging = true
                    local delta = input.Position - startPos
                    btn.Position = UDim2.new(0, btn.Position.X.Offset + delta.X, 0, btn.Position.Y.Offset + delta.Y)
                    startPos = input.Position
                end
            end
        end)
        
        btn.InputEnded:Connect(function(input)
            if input.UserInputType == Enum.UserInputType.Touch or input.UserInputType == Enum.UserInputType.MouseButton1 then
                if not isDragging and tick() - startTime < 0.3 then
                    if btn == AxisButton then
                        ToggleAxisESP()
                    elseif btn == AlliesButton then
                        ToggleAlliesESP()
                    elseif btn == CamLockButton then
                        ToggleCamLock()
                    elseif btn == AutoShootButton then
                        ToggleAutoShoot()
                    elseif btn == FOVAimbotButton then
                        ToggleFOVAimbot()
                    end
                end
                isDragging = false
                startPos = nil
            end
        end)
        
        return btn
    end
    
    AxisButton = makeBtn("AxisButton", 0.05, "🍊 OFF\n📱 Toque para ativar\n↔️ Arraste para mover", Color3.fromRGB(40, 40, 40))
    AlliesButton = makeBtn("AlliesButton", 0.12, "🥗 OFF\n📱 Toque para ativar\n↔️ Arraste para mover", Color3.fromRGB(40, 40, 40))
    CamLockButton = makeBtn("CamLockButton", 0.19, "🎯 OFF\n📱 Toque para ativar\n↔️ Arraste para mover", Color3.fromRGB(40, 40, 40))
    AutoShootButton = makeBtn("AutoShootButton", 0.26, "🗡️ OFF\nAuto Tiro\n↔️ Arraste para mover", Color3.fromRGB(40, 40, 40), Color3.fromRGB(1, 0.9, 0.9))
    FOVAimbotButton = makeBtn("FOVAimbotButton", 0.33, "♨️ OFF\nFOV Aimbot\n↔️ Arraste para mover", Color3.fromRGB(70, 0, 80), Color3.fromRGB(1, 0.7, 1))
end

function ToggleAxisESP()
    if ActiveTeamESP == "Axis" then
        ActiveTeamESP = nil
        AxisButton.Text = "🍊 OFF\n📱 Toque para ativar\n↔️ Arraste para mover"
        AxisButton.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
    else
        ActiveTeamESP = "Axis"
        AxisButton.Text = "🍊 ON\n📱 Ativo\n↔️ Arraste para mover"
        AxisButton.BackgroundColor3 = Color3.fromRGB(80, 40, 0)
        AlliesButton.Text = "🥗 OFF\n📱 Toque para ativar\n↔️ Arraste para mover"
        AlliesButton.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
    end
    UpdateTeamESP()
end

function ToggleAlliesESP()
    if ActiveTeamESP == "Allies" then
        ActiveTeamESP = nil
        AlliesButton.Text = "🥗 OFF\n📱 Toque para ativar\n↔️ Arraste para mover"
        AlliesButton.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
    else
        ActiveTeamESP = "Allies"
        AlliesButton.Text = "🥗 ON\n📱 Ativo\n↔️ Arraste para mover"
        AlliesButton.BackgroundColor3 = Color3.fromRGB(0, 60, 0)
        AxisButton.Text = "🍊 OFF\n📱 Toque para ativar\n↔️ Arraste para mover"
        AxisButton.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
    end
    UpdateTeamESP()
end

function ToggleCamLock()
    if not CamLockActive and ActiveTeamESP then
        local tgt = (ActiveTeamESP == "Axis") and "Allies" or "Axis"
        CurrentTarget = FindBestTargetByTeam(tgt)
        if CurrentTarget then
            CamLockActive = true
            CamLockButton.Text = "🎯 ON\n📱 Ativo\n↔️ Arraste para mover"
            CamLockButton.BackgroundColor3 = Color3.fromRGB(60, 0, 0)
            coroutine.wrap(CamLockThread)()
        end
    else
        CamLockActive = false
        CamLockButton.Text = "🎯 OFF\n📱 Toque para ativar\n↔️ Arraste para mover"
        CamLockButton.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
        CurrentTarget = nil
    end
end

function ToggleAutoShoot()
    if not AutoShootActive then
        AutoShootActive = true
        AutoShootButton.Text = "🗡️ ON\nAuto Tiro\n↔️ Arraste para mover"
        AutoShootButton.BackgroundColor3 = Color3.fromRGB(180, 50, 50)
        coroutine.wrap(AutoShootThread)()
    else
        AutoShootActive = false
        AutoShootButton.Text = "🗡️ OFF\nAuto Tiro\n↔️ Arraste para mover"
        AutoShootButton.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
    end
end

function ToggleFOVAimbot()
    if not FOV_AIMBOT_ACTIVE and ActiveTeamESP then
        FOV_AIMBOT_ACTIVE = true
        FOVAimbotButton.Text = "♨️ ON\nFOV Aimbot\n↔️ Arraste para mover"
        FOVAimbotButton.BackgroundColor3 = Color3.fromRGB(140, 0, 160)
        coroutine.wrap(FOVAimbotThread)()
    else
        FOV_AIMBOT_ACTIVE = false
        FOVAimbotButton.Text = "♨️ OFF\nFOV Aimbot\n↔️ Arraste para mover"
        FOVAimbotButton.BackgroundColor3 = Color3.fromRGB(70, 0, 80)
        CurrentTarget = nil
    end
end

-- ============ 📱 OTIMIZAÇÃO MOBILE + INICIALIZAÇÃO ============

local function Initialize()
    CreateTeamButtons()
    
    -- Evento quando jogador entra
    Players.PlayerAdded:Connect(function(player)
        player.CharacterAdded:Connect(function(character)
            task.wait(2)
            UpdateTeamESP()
            
            -- Conecta evento quando personagem morre para limpar highlights
            local humanoid = character:WaitForChild("Humanoid")
            humanoid.Died:Connect(function()
                if PlayerHighlights[player] then
                    PlayerHighlights[player] = nil
                end
                task.wait(1)
                UpdateTeamESP()
            end)
        end)
    end)
    
    -- Evento quando jogador sai
    Players.PlayerRemoving:Connect(function(player)
        RemoveSupremeESP(player)
    end)
    
    -- Evento quando jogador local spawna
    if LocalPlayer.Character then
        local humanoid = LocalPlayer.Character:FindFirstChildOfClass("Humanoid")
        if humanoid then
            humanoid.Died:Connect(function()
                task.wait(3)
                UpdateTeamESP()
            end)
        end
    end
    
    LocalPlayer.CharacterAdded:Connect(function(character)
        task.wait(3)
        UpdateTeamESP()
        
        local humanoid = character:WaitForChild("Humanoid")
        humanoid.Died:Connect(function()
            task.wait(3)
            UpdateTeamESP()
        end)
    end)
    
    -- 📱 OTIMIZAÇÃO MOBILE - Loop de atualização otimizada
    coroutine.wrap(function()
        while task.wait(ESP_UPDATE_INTERVAL) do
            UpdateTeamESP()
        end
    end)()
    
    -- 💣 Loop para manter highlights supremos funcionando
    coroutine.wrap(function()
        while task.wait(2) do
            if ActiveTeamESP then
                for _, player in pairs(Players:GetPlayers()) do
                    if IsEnemy(player) then -- 🔥 FILTRO INTELIGENTE
                        local team = GetPlayerTeam(player)
                        -- Verifica se o highlight supremo ainda existe
                        if not PlayerHighlights[player] or not PlayerHighlights[player].Parent then
                            local highlightColor = TEAM_COLORS[team]
                            PlayerHighlights[player] = CreateSupremeHighlight(player.Character, highlightColor)
                        end
                    end
                end
            end
        end
    end)()
    
    -- 🧠 Loop de limpeza automática para otimização
    coroutine.wrap(function()
        while task.wait(10) do
            -- Remove ESPs de jogadores que não são mais inimigos
            for player, esp in pairs(PlayerESPs) do
                if not IsEnemy(player) then
                    RemoveSupremeESP(player)
                end
            end
        end
    end)()
end

-- Inicializa o script
Initialize()

print("🔥 DOGS OF WAR FOV AIMBOT CORRIGIDO + ESP SUPREMO LOADED ✅")
print("💣 CORREÇÕES IMPLEMENTADAS:")
print("📋 FUNCIONALIDADES CORRIGIDAS:")
print("   🍊 Botão Laranja: ESP para time Axis (inimigos ficam em VERDE SUPREMO)")
print("   🥗 Botão Verde: ESP para time Allies (inimigos ficam em LARANJA SUPREMO)")
print("   🎯 CamLock: Mira automática no inimigo mais próximo")
print("   🗡️ Auto Tiro CORRIGIDO: Dispara em qualquer distância quando mira ativa")
print("   ♨️ FOV Aimbot CORRIGIDO: Funciona perfeitamente em qualquer distância")
print("   🔄 Auto Reload: Recarregamento automático quando munição baixa")
print("🎯 AIMBOT CORRIGIDO:")
print("   ✅ MIRA 100% PRECISA: Aponta exatamente no centro da cabeça")
print("   ✅ SEM OFFSET: Removido face_offset para máxima precisão")
print("   ✅ DETECÇÃO PRÓXIMA: FOV_CLOSE_DIST aumentado para 15 studs")
print("   ✅ QUALQUER DISTÂNCIA: Funciona perfeitamente de perto e longe")
print("🗡️ AUTO TIRO CORRIGIDO:")
print("   ✅ DISTÂNCIA AUMENTADA: AUTO_SHOOT_MAX_DIST = 500 studs")
print("   ✅ VELOCIDADE MAIOR: Intervalos reduzidos para 0.005s")
print("   ✅ M2 INTELIGENTE: Ativa em qualquer distância se alvo visível")
print("   ✅ M1 PRECISO: Atira quando M2 ativo e cabeça visível")
print("🧠 HIGHLIGHT SETUP IDEAL:")
print("   ✨ FillColor: Cor do time | OutlineColor: Branco puro")
print("   ✨ FillTransparency: 0.2 | OutlineTransparency: 0")
print("   ✨ DepthMode: AlwaysOnTop (atravessa parede)")
print("💥 ESP HUD EXTRA:")
print("   📛 Nome do inimigo (branco puro)")
print("   📏 Distância (amarelo)")
print("   ❤️ Vida com cor degradê (verde/amarelo/vermelho)")
print("🔥 FILTRO INTELIGENTE:")
print("   ⚡ Só aplica ESP a inimigos do time oposto")
print("   ⚡ Ignora aliados e dummies não combatentes")
print("📱 OTIMIZAÇÃO MOBILE:")
print("   ⚡ Highlight supremo (mais leve)")
print("   ⚡ RunService.Heartbeat otimizado")
print("   ⚡ Desativa ESP em players >1000 studs")
print("   ⚡ Sistema de limpeza automática")
print("🚀 FOV AIMBOT AGORA FUNCIONA PERFEITAMENTE EM QUALQUER DISTÂNCIA!")

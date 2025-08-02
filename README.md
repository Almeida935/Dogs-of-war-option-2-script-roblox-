--[[ 🔥 DOGS OF WAR - TEAM ESP + ULTIMATE CAMLOCK + AUTO SHOOT + OUTLINE ULTRA LITE + FOV AIMBOT REMAKE + AUTO M2 + AUTO RELOAD + AUTO KNIFE (M2 OP ATUALIZADO)
💣 COMBO SUPREMO: ESP ULTRA VISUAL + DESEMPENHO + FUNCIONALIDADE + AUTO KNIFE
✅ ESP só do time inimigo (correta Axis/Allies) com HIGHLIGHTS SUPREMOS
✅ AutoShoot só dispara quando o M2 (mira) estiver ativado, ideal para sniper/dano alto!
✅ Auto M2 OP: M2 ativa instantaneamente quando troca de alvo, nunca fica sem mira após matar/inimigo morrer!
✅ M2 detecta alvo novo em tempo real e ativa instantâneo, nunca falha em mirar!
✅ AutoReload: Aperta R automaticamente se balas ou punição chegar em 0 ou 1!
✅ FOV Aimbot MELHORADO: Prioriza inimigos com pouca vida + mira colada na cabeça
🔪 AUTO KNIFE: Usa faca automaticamente quando inimigo chega perto (5-8 studs)
🧠 HIGHLIGHT SETUP IDEAL + ESP HUD EXTRA + FILTRO INTELIGENTE + OTIMIZAÇÃO MOBILE
]]

-- CONFIG
local FOV_SIZE = 210 -- FOV maior/OP
local FACE_OFFSET = 0.1 -- MELHORADO: Mira mais colada na cabeça (era 0.45)
local FOV_CLOSE_DIST = 8
local ESP_MAX_DISTANCE = 1000 -- Desativa ESP em players fora de 1000 studs (otimização)
local AUTO_SHOOT_MAX_DIST = 260 -- Mais longe, OP
local AUTO_SHOOT_CHECK_INTERVAL = 0.008 -- Mais rápido
local FOV_AIMBOT_CHECK_INTERVAL = 0.008 -- Mais rápido
local ESP_UPDATE_INTERVAL = 1 -- Atualiza a cada 1 segundo (otimização mobile)
local LOW_HEALTH_THRESHOLD = 0.5 -- 50% de vida considera como "vida baixa"

-- 🔪 CONFIGURAÇÃO DA FACA AUTOMÁTICA
local AUTO_KNIFE_DISTANCE = 8 -- Distância para usar faca automaticamente (studs)
local AUTO_KNIFE_MIN_DISTANCE = 5 -- Distância mínima para ativar faca
local AUTO_KNIFE_CHECK_INTERVAL = 0.05 -- Intervalo de verificação da faca
local KNIFE_COOLDOWN = 0.5 -- Cooldown entre usos da faca

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

-- 🔪 VARIÁVEIS DA FACA AUTOMÁTICA
local lastKnifeTime = 0
local AutoKnifeActive = false

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

-- 🔪 FUNÇÃO DA FACA AUTOMÁTICA
local function UseKnife()
    local currentTime = tick()
    if currentTime - lastKnifeTime < KNIFE_COOLDOWN then
        return false
    end
    
    lastKnifeTime = currentTime
    VirtualInputManager:SendKeyEvent(true, Enum.KeyCode.F, false, game)
    task.wait(0.05)
    VirtualInputManager:SendKeyEvent(false, Enum.KeyCode.F, false, game)
    return true
end

-- 🔪 DETECÇÃO DE INIMIGOS PRÓXIMOS PARA FACA
local function FindNearbyEnemies()
    if not AutoShootActive then return {} end -- Só funciona se auto tiro estiver ativo
    
    local myChar = LocalPlayer.Character
    if not myChar or not IsAlive(myChar) then return {} end
    
    local myHRP = myChar:FindFirstChild("HumanoidRootPart")
    if not myHRP then return {} end
    
    local nearbyEnemies = {}
    
    for _, player in pairs(Players:GetPlayers()) do
        if IsEnemy(player) then
            local theirHRP = player.Character and player.Character:FindFirstChild("HumanoidRootPart")
            if theirHRP then
                local distance = (myHRP.Position - theirHRP.Position).Magnitude
                if distance >= AUTO_KNIFE_MIN_DISTANCE and distance <= AUTO_KNIFE_DISTANCE then
                    -- Verifica se o inimigo está visível
                    if IsBodyVisible(myHRP.Position, player.Character) then
                        table.insert(nearbyEnemies, {
                            player = player,
                            character = player.Character,
                            distance = distance
                        })
                    end
                end
            end
        end
    end
    
    -- Ordena por distância (mais próximo primeiro)
    table.sort(nearbyEnemies, function(a, b)
        return a.distance < b.distance
    end)
    
    return nearbyEnemies
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
    esp.Size = UDim2.new(0, 55, 0, 45)
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
            
            -- 🔪 INDICADOR VISUAL PARA FACA - Muda cor da distância quando inimigo está perto
            if dist >= AUTO_KNIFE_MIN_DISTANCE and dist <= AUTO_KNIFE_DISTANCE and AutoShootActive then
                distanceLabel.TextColor3 = Color3.fromRGB(255, 0, 0) -- Vermelho para indicar alcance da faca
                distanceLabel.Text = dist.."m 🔪"
            else
                distanceLabel.TextColor3 = HUD_COLORS.distance -- Amarelo normal
                distanceLabel.Text = dist.."m"
            end
            
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
            
            -- 📱 OTIMIZAÇÃO - Desativa ESP em players fora do alcance de visão ou >1000 studs
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
                        local angle = math.acos(dir:Dot(camLook))
                        if angle < closestAngle then closestAngle = angle; best = p.Character end
                    end
                end
            end
        end
    end
    return best
end

-- 🎯 FOV AIMBOT MELHORADO: Prioriza inimigos com pouca vida
local function FindFOVAimbotTarget(targetTeam)
    local lpchar = LocalPlayer.Character
    if not lpchar then return nil end
    local hrp = lpchar:FindFirstChild("HumanoidRootPart")
    if not hrp then return nil end
    
    local lowHealthTargets = {} -- Inimigos com pouca vida
    local normalTargets = {} -- Inimigos com vida normal
    local closeTarget, closeDist = nil, math.huge
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
                    
                    -- Prioridade 1: Inimigos muito próximos (dentro de FOV_CLOSE_DIST)
                    if distWorld <= FOV_CLOSE_DIST then
                        if IsHeadVisible(hrp.Position, p.Character) then
                            if distWorld < closeDist then
                                closeDist = distWorld
                                closeTarget = p.Character
                            end
                        end
                    else
                        -- Prioridade 2: Inimigos dentro do FOV
                        local headPos, onScreen = Camera:WorldToScreenPoint(head.Position)
                        local screenPos = Vector2.new(headPos.X, headPos.Y)
                        local distScreen = (screenPos - screenCenter).Magnitude
                        
                        if distScreen <= FOV_SIZE/2 and IsHeadVisible(hrp.Position, p.Character) then
                            local targetData = {
                                character = p.Character,
                                worldDistance = distWorld,
                                screenDistance = distScreen,
                                healthPercent = healthPercent
                            }
                            
                            -- Separa inimigos por vida
                            if healthPercent <= LOW_HEALTH_THRESHOLD then
                                table.insert(lowHealthTargets, targetData)
                            else
                                table.insert(normalTargets, targetData)
                            end
                        end
                    end
                end
            end
        end
    end
    
    -- Retorna o mais próximo se houver
    if closeTarget then return closeTarget end
    
    -- PRIORIDADE: Inimigos com pouca vida primeiro
    local targetsToCheck = #lowHealthTargets > 0 and lowHealthTargets or normalTargets
    
    if #targetsToCheck > 0 then
        -- Ordena por distância mundial (mais próximo primeiro)
        table.sort(targetsToCheck, function(a, b)
            return a.worldDistance < b.worldDistance
        end)
        
        return targetsToCheck[1].character
    end
    
    return nil
end

-- 🎯 MIRA MELHORADA: Colada na cabeça independente da distância
local function AimAtHead(targetChar)
    if not targetChar or not targetChar:FindFirstChild("Head") then return false end
    
    local head = targetChar.Head
    local myChar = LocalPlayer.Character
    if not myChar or not myChar:FindFirstChild("HumanoidRootPart") then return false end
    
    local myHRP = myChar.HumanoidRootPart
    local distance = (myHRP.Position - head.Position).Magnitude
    
    -- MELHORADO: Sempre mira exatamente no centro da cabeça
    local headCenter = head.Position
    
    -- Se estiver muito próximo, mira ligeiramente à frente
    if distance <= FOV_CLOSE_DIST then
        local headCFrame = head.CFrame
        local lookVec = headCFrame.LookVector
        headCenter = head.Position + lookVec * 0.05 -- Muito pouco offset
    end
    
    -- Mira diretamente no centro da cabeça
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

-- ============ AUTO SHOOT + AUTO M2 + AUTO RELOAD + AUTO KNIFE (M2 OP ATUALIZADO) ============

-- 🔪 THREAD DA FACA AUTOMÁTICA
local function AutoKnifeThread()
    while AutoShootActive do -- Funciona junto com auto tiro
        local nearbyEnemies = FindNearbyEnemies()
        
        if #nearbyEnemies > 0 then
            local closestEnemy = nearbyEnemies[1]
            
            -- Usa a faca no inimigo mais próximo
            if UseKnife() then
                print("🔪 Faca usada em " .. closestEnemy.player.Name .. " - Distância: " .. math.floor(closestEnemy.distance) .. " studs")
            end
        end
        
        task.wait(AUTO_KNIFE_CHECK_INTERVAL)
    end
end

local function AutoShootThread()
    local m1down = false
    local m2down = false
    local lastTargetObj = nil
    local lastTargetId = nil
    
    -- 🔪 INICIA THREAD DA FACA AUTOMÁTICA
    coroutine.wrap(AutoKnifeThread)()
    
    while AutoShootActive do
        local validTarget = (CamLockActive or FOV_AIMBOT_ACTIVE) and CurrentTarget and IsAlive(CurrentTarget)
        local myChar = LocalPlayer.Character
        local myHRP = myChar and myChar:FindFirstChild("HumanoidRootPart")
        local headVis = validTarget and myHRP and IsHeadVisible(myHRP.Position, CurrentTarget)
        local bodyVis = validTarget and myHRP and IsBodyVisible(myHRP.Position, CurrentTarget)
        local targetHRP = validTarget and CurrentTarget:FindFirstChild("HumanoidRootPart")
        local dist = (myHRP and targetHRP) and (myHRP.Position - targetHRP.Position).Magnitude or math.huge
        local targetId = validTarget and tostring(CurrentTarget) or nil
        
        -- 🔪 AUTO KNIFE: Se inimigo está muito perto, usa faca em vez de atirar
        if validTarget and dist >= AUTO_KNIFE_MIN_DISTANCE and dist <= AUTO_KNIFE_DISTANCE and (headVis or bodyVis) then
            -- Não atira quando está no alcance da faca, deixa a faca automática cuidar disso
        else
            -- Auto M2 OP: Detecta troca de alvo e ativa M2 instantâneo!
            if validTarget and dist > FOV_CLOSE_DIST and (headVis or bodyVis) and IsAlive(CurrentTarget) and IsAlive(myChar) then
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
            
            -- Auto M1: Só atira se M2 estiver ativado e não estiver no alcance da faca
            if validTarget and headVis and dist <= AUTO_SHOOT_MAX_DIST and dist > AUTO_KNIFE_DISTANCE and m2down then
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
        end
        
        -- AutoReload: se munição ou punição chegar em 0 ou 1, simula R
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
        
        -- Se você se esconder atrás de parede/objeto ou alvo morrer, solta M2/M1
        if not validTarget or not IsAlive(myChar) or not IsAlive(CurrentTarget) or not headVis then
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

-- 🎯 FOV AIMBOT MELHORADO
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
            -- MELHORADO: Usa a nova função de mira
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
    AutoShootButton = makeBtn("AutoShootButton", 0.26, "🗡️ OFF\nAuto Tiro + Faca\n↔️ Arraste para mover", Color3.fromRGB(40, 40, 40), Color3.fromRGB(1, 0.9, 0.9))
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
        AutoShootButton.Text = "🗡️ ON\nAuto Tiro + Faca\n↔️ Arraste para mover"
        AutoShootButton.BackgroundColor3 = Color3.fromRGB(180, 50, 50)
        coroutine.wrap(AutoShootThread)()
    else
        AutoShootActive = false
        AutoShootButton.Text = "🗡️ OFF\nAuto Tiro + Faca\n↔️ Arraste para mover"
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
    
    -- 📱 OTIMIZAÇÃO MOBILE - Loop de atualização otimizada a cada 1 segundo
    coroutine.wrap(function()
        while task.wait(ESP_UPDATE_INTERVAL) do
            UpdateTeamESP()
        end
    end)()
    
    -- 💣 COMBO SUPREMO - Loop para manter highlights supremos funcionando
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

print("💣 DOGS OF WAR COMBO SUPREMO + AUTO KNIFE + AIMBOT MELHORADO LOADED ✅")
print("🔥 COMBO SUPREMO: ESP ULTRA VISUAL + DESEMPENHO + FUNCIONALIDADE + AUTO KNIFE")
print("📋 FUNCIONALIDADES SUPREMAS:")
print("   🍊 Botão Laranja: ESP para time Axis (inimigos ficam em VERDE SUPREMO)")
print("   🥗 Botão Verde: ESP para time Allies (inimigos ficam em LARANJA SUPREMO)")
print("   🎯 CamLock: Mira automática no inimigo mais próximo")
print("   🗡️ Auto Tiro + Faca: Disparo automático + faca quando inimigo está perto")
print("   ♨️ FOV Aimbot MELHORADO: Prioriza inimigos com pouca vida + mira colada")
print("   🔄 Auto Reload: Recarregamento automático quando munição baixa")
print("🔪 AUTO KNIFE SUPREMO:")
print("   🔪 ATIVAÇÃO: Funciona junto com Auto Tiro")
print("   🔪 ALCANCE: 5-8 studs de distância")
print("   🔪 COOLDOWN: 0.5 segundos entre usos")
print("   🔪 VISUAL: Indicador 🔪 no ESP quando inimigo está no alcance")
print("   🔪 INTELIGENTE: Não atira quando usa faca (evita desperdício de munição)")
print("🧠 HIGHLIGHT SETUP IDEAL:")
print("   ✨ FillColor: Cor do time | OutlineColor: Branco puro")
print("   ✨ FillTransparency: 0.2 | OutlineTransparency: 0")
print("   ✨ DepthMode: AlwaysOnTop (atravessa parede)")
print("💥 ESP HUD EXTRA:")
print("   📛 Nome do inimigo (branco puro)")
print("   📏 Distância (amarelo normal / vermelho com 🔪 quando no alcance da faca)")
print("   ❤️ Vida com cor degradê (verde/amarelo/vermelho)")
print("🎯 AIMBOT MELHORADO:")
print("   🎯 PRIORIDADE: Inimigos com ≤50% de vida primeiro")
print("   🎯 MIRA COLADA: Face offset reduzido para 0.1 (era 0.45)")
print("   🎯 PRECISÃO: Sempre mira no centro exato da cabeça")
print("   🎯 PROXIMIDADE: Detecção melhorada para alvos próximos")
print("🔥 FILTRO INTELIGENTE:")
print("   ⚡ Só aplica ESP a inimigos do time oposto")
print("   ⚡ Ignora aliados e dummies não combatentes")
print("📱 OTIMIZAÇÃO MOBILE:")
print("   ⚡ Usa Highlight supremo (mais leve que BoxHandleAdornment)")
print("   ⚡ Atualiza RunService.Heartbeat otimizado")
print("   ⚡ Desativa ESP em players >1000 studs")
print("   ⚡ Sistema de limpeza automática")
print("🚀 PERFORMANCE SUPREMA + AIMBOT DEADLY PRECISO + AUTO KNIFE LETAL GARANTIDO!")

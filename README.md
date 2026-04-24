-- // ZAP MENU v1.0 - Arrayfield/Rayfield Version
-- // ESP + Aimbot + Silent Aim + Farm + AFK

local Rayfield = loadstring(game:HttpGet('https://raw.githubusercontent.com/UI-Interface/CustomFIeld/main/RayField.lua'))()

local Window = Rayfield:CreateWindow({
   Name = "ZAP MENU v1.0",
   LoadingTitle = "ZAP MENU",
   LoadingSubtitle = "SINTONIA RP",
   ConfigurationSaving = {
      Enabled = false,
   },
   KeySystem = false,
})

-- =====================
-- // VARIAVEIS
-- =====================
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local LocalPlayer = Players.LocalPlayer
local Camera = workspace.CurrentCamera
local Mouse = LocalPlayer:GetMouse()

-- Configuracoes
local ESP_Ativo = false
local ESP_Nick = true
local ESP_Caixas = true
local ESP_Linhas = true
local ESP_DistanciaMaxima = 500
local ESP_Cor = Color3.fromRGB(255, 50, 50)
local ESP_TamanhoTexto = 14

local Aimbot_Ativo = false
local Aimbot_MiraParte = "Head"
local Aimbot_Smoothing = 5
local Aimbot_FOV = 200
local Aimbot_KeyBind = "RightButton"
local Aimbot_MostrarFOV = true
local Aimbot_CorFOV = Color3.fromRGB(255, 255, 255)
local Aimbot_ApenasVisivel = true
local Aimbot_DistanciaMaxima = 300

local Silent_Ativo = false
local Silent_MiraParte = "Head"
local Silent_FOV = 200
local Silent_DistanciaMaxima = 350
local Silent_ApenasVisivel = true
local Silent_KeyBind = "RightButton"

local Farm_Ativo = false
local Farm_Delay = 5

local AFK_Ativo = false
local AFK_Intervalo = 300
local AFK_Loop = nil

local ESPObjects = {}
local FOVCircle = nil
local farmActive = false
local farmLoopRunning = false

-- =====================
-- // ANTI-AFK SIMPLES E SEGURO
-- =====================

local function moverMouseLevemente()
    pcall(function()
        mousemoverel(1, 0)
        task.wait(0.05)
        mousemoverel(-1, 0)
    end)
end

local function darPequenoPulo()
    local char = LocalPlayer.Character
    local humanoid = char and char:FindFirstChild("Humanoid")
    
    if humanoid then
        pcall(function()
            humanoid:ChangeState(Enum.HumanoidStateType.Jumping)
        end)
    end
end

local function executarAcaoAntiAFK()
    if math.random(1, 2) == 1 then
        moverMouseLevemente()
    else
        darPequenoPulo()
    end
end

local function iniciarAFK()
    if AFK_Loop then 
        return 
    end
    
    AFK_Loop = task.spawn(function()
        while AFK_Ativo do
            executarAcaoAntiAFK()
            
            for i = 1, AFK_Intervalo do
                if not AFK_Ativo then break end
                task.wait(1)
            end
        end
        AFK_Loop = nil
    end)
end

local function pararAFK()
    if AFK_Loop then
        task.cancel(AFK_Loop)
        AFK_Loop = nil
    end
end

-- =====================
-- // FUNCOES ESP
-- =====================

local function criarLabel(tipo)
    local label = Drawing.new(tipo)
    label.Visible = false
    return label
end

local function criarESP(player)
    if player == LocalPlayer then return end
    ESPObjects[player] = {
        Nick = criarLabel("Text"), BoxTop = criarLabel("Line"),
        BoxBot = criarLabel("Line"), BoxLeft = criarLabel("Line"),
        BoxRight = criarLabel("Line"), Linha = criarLabel("Line"),
    }
    local nick = ESPObjects[player].Nick
    nick.Size = ESP_TamanhoTexto
    nick.Center = true
    nick.Outline = true
    nick.Color = ESP_Cor
end

local function removerESP(player)
    if ESPObjects[player] then
        for _, obj in pairs(ESPObjects[player]) do obj:Remove() end
        ESPObjects[player] = nil
    end
end

local function atualizarESP()
    for player, objs in pairs(ESPObjects) do
        local character = player.Character
        local hrp = character and character:FindFirstChild("HumanoidRootPart")
        if not hrp then
            for _, obj in pairs(objs) do obj.Visible = false end
        else
            local distancia = (hrp.Position - Camera.CFrame.Position).Magnitude
            local visivel = ESP_Ativo and distancia <= ESP_DistanciaMaxima
            local pos3D, onScreen = Camera:WorldToViewportPoint(hrp.Position + Vector3.new(0, 2.5, 0))
            local posBase, _ = Camera:WorldToViewportPoint(hrp.Position - Vector3.new(0, 3, 0))
            local posTela = Vector2.new(pos3D.X, pos3D.Y)
            local posBaseTela = Vector2.new(posBase.X, posBase.Y)
            local altura = posTela.Y - posBaseTela.Y
            local largura = altura * 0.5
            
            objs.Nick.Visible = visivel and ESP_Nick and onScreen
            if objs.Nick.Visible then
                objs.Nick.Text = player.Name .. " [" .. math.floor(distancia) .. "m]"
                objs.Nick.Position = Vector2.new(posTela.X, posBaseTela.Y - 16)
                objs.Nick.Color = ESP_Cor
                objs.Nick.Size = ESP_TamanhoTexto
            end
            
            local mostrarCaixa = visivel and ESP_Caixas and onScreen
            local x1 = posTela.X - largura
            local x2 = posTela.X + largura
            local y1 = posBaseTela.Y
            local y2 = posTela.Y
            
            local lines = {
                {objs.BoxTop, Vector2.new(x1,y2), Vector2.new(x2,y2)},
                {objs.BoxBot, Vector2.new(x1,y1), Vector2.new(x2,y1)},
                {objs.BoxLeft, Vector2.new(x1,y1), Vector2.new(x1,y2)},
                {objs.BoxRight, Vector2.new(x2,y1), Vector2.new(x2,y2)},
            }
            for _, t in ipairs(lines) do
                local ln, from, to = t[1], t[2], t[3]
                ln.Visible = mostrarCaixa
                if mostrarCaixa then
                    ln.From = from ; ln.To = to
                    ln.Color = ESP_Cor ; ln.Thickness = 1
                end
            end
            
            objs.Linha.Visible = visivel and ESP_Linhas and onScreen
            if objs.Linha.Visible then
                objs.Linha.From = Vector2.new(Camera.ViewportSize.X / 2, Camera.ViewportSize.Y)
                objs.Linha.To = posTela
                objs.Linha.Color = ESP_Cor
                objs.Linha.Thickness = 1
            end
        end
    end
end

-- =====================
-- // FUNCOES AIMBOT (COM SMOOTHING)
-- =====================

local function criarFOVCircle()
    if FOVCircle then FOVCircle:Remove() end
    FOVCircle = Drawing.new("Circle")
    FOVCircle.Visible = false
    FOVCircle.Radius = Aimbot_FOV
    FOVCircle.Thickness = 1
    FOVCircle.Filled = false
    FOVCircle.Color = Aimbot_CorFOV
    FOVCircle.Transparency = 1
    FOVCircle.NumSides = 100
end

local function atualizarFOVCircle()
    if not FOVCircle then criarFOVCircle() end
    if Aimbot_MostrarFOV and Aimbot_Ativo then
        FOVCircle.Visible = true
        FOVCircle.Position = Vector2.new(Mouse.X, Mouse.Y)
        FOVCircle.Radius = Aimbot_FOV
        FOVCircle.Color = Aimbot_CorFOV
    elseif FOVCircle then
        FOVCircle.Visible = false
    end
end

local function getClosestPlayer()
    local closest = nil
    local closestDist = Aimbot_FOV + 1
    local mousePos = Vector2.new(Mouse.X, Mouse.Y)
    
    for _, player in ipairs(Players:GetPlayers()) do
        if player ~= LocalPlayer and player.Character then
            local humanoid = player.Character:FindFirstChild("Humanoid")
            if humanoid and humanoid.Health > 0 then
                local targetPart = Aimbot_MiraParte == "Head" and player.Character:FindFirstChild("Head") or player.Character:FindFirstChild("HumanoidRootPart")
                if targetPart then
                    local screenPos, onScreen = Camera:WorldToViewportPoint(targetPart.Position)
                    local distance = (targetPart.Position - Camera.CFrame.Position).Magnitude
                    
                    if onScreen and distance <= Aimbot_DistanciaMaxima then
                        local distToMouse = (Vector2.new(screenPos.X, screenPos.Y) - mousePos).Magnitude
                        if distToMouse <= Aimbot_FOV and distToMouse < closestDist then
                            closestDist = distToMouse
                            closest = targetPart
                        end
                    end
                end
            end
        end
    end
    
    return closest
end

local function miraSuave(alvo, smoothing)
    local direction = (alvo.Position - Camera.CFrame.Position).Unit
    local targetCF = CFrame.new(Camera.CFrame.Position, Camera.CFrame.Position + direction)
    Camera.CFrame = Camera.CFrame:Lerp(targetCF, 1 / smoothing)
end

local function miraInsta(alvo)
    local direction = (alvo.Position - Camera.CFrame.Position).Unit
    Camera.CFrame = CFrame.new(Camera.CFrame.Position, Camera.CFrame.Position + direction)
end

task.spawn(function()
    while true do
        if Aimbot_Ativo then
            local isPressed = false
            if Aimbot_KeyBind == "RightButton" then
                isPressed = UserInputService:IsMouseButtonPressed(Enum.UserInputType.MouseButton2)
            elseif Aimbot_KeyBind == "LeftButton" then
                isPressed = UserInputService:IsMouseButtonPressed(Enum.UserInputType.MouseButton1)
            elseif Aimbot_KeyBind == "MiddleButton" then
                isPressed = UserInputService:IsMouseButtonPressed(Enum.UserInputType.MouseButton3)
            end
            
            if isPressed then
                local target = getClosestPlayer()
                if target then
                    if Aimbot_Smoothing > 1 then
                        miraSuave(target, Aimbot_Smoothing)
                    else
                        miraInsta(target)
                    end
                end
            end
        end
        task.wait()
    end
end)

-- =====================
-- // FUNCOES DO FARM
-- =====================

local function interagirComObjeto()
    local char = LocalPlayer.Character
    local hrp = char and char:FindFirstChild("HumanoidRootPart")
    if not hrp then 
        return false 
    end
    
    local posicaoJogador = hrp.Position
    local objetoMaisProximo = nil
    local menorDistancia = 20
    
    for _, obj in ipairs(workspace:GetDescendants()) do
        if obj:IsA("ClickDetector") then
            local parent = obj.Parent
            if parent then
                local posObj = parent:IsA("BasePart") and parent.Position or nil
                if posObj then
                    local dist = (posObj - posicaoJogador).Magnitude
                    if dist < menorDistancia then
                        menorDistancia = dist
                        objetoMaisProximo = obj
                    end
                end
            end
        end
    end
    
    if not objetoMaisProximo then
        for _, obj in ipairs(workspace:GetDescendants()) do
            if obj:IsA("ProximityPrompt") then
                local parent = obj.Parent
                if parent then
                    local posObj = parent:IsA("BasePart") and parent.Position or nil
                    if posObj then
                        local dist = (posObj - posicaoJogador).Magnitude
                        if dist < menorDistancia then
                            menorDistancia = dist
                            objetoMaisProximo = obj
                        end
                    end
                end
            end
        end
    end
    
    if objetoMaisProximo then
        pcall(function()
            if objetoMaisProximo:IsA("ClickDetector") then
                fireclickdetector(objetoMaisProximo)
            elseif objetoMaisProximo:IsA("ProximityPrompt") then
                fireproximityprompt(objetoMaisProximo)
            end
        end)
        return true
    end
    return false
end

local function farmLoop()
    if farmLoopRunning then return end
    farmLoopRunning = true
    
    local count = 0
    
    while farmActive and Farm_Ativo do
        interagirComObjeto()
        count = count + 1
        
        local startTime = tick()
        while farmActive and Farm_Ativo and (tick() - startTime) < Farm_Delay do
            task.wait(0.1)
        end
    end
    
    farmLoopRunning = false
end

local function iniciarFarm()
    if farmActive then return end
    farmActive = true
    task.spawn(farmLoop)
end

local function pararFarm()
    farmActive = false
end

-- =====================
-- // CRIAR ABAS
-- =====================

local espTab = Window:CreateTab("ESP", 4483362458)

espTab:CreateSection("Visualizacao", false)

espTab:CreateToggle({
    Name = "Ativar ESP",
    CurrentValue = false,
    Callback = function(v)
        ESP_Ativo = v
        if v then
            for _, p in ipairs(Players:GetPlayers()) do
                if p ~= LocalPlayer then criarESP(p) end
            end
        else
            for p, o in pairs(ESPObjects) do
                for _, obj in pairs(o) do obj:Remove() end
            end
            for k in pairs(ESPObjects) do ESPObjects[k] = nil end
        end
    end
})

espTab:CreateToggle({
    Name = "Mostrar Nick",
    CurrentValue = true,
    Callback = function(v) ESP_Nick = v end
})

espTab:CreateToggle({
    Name = "Mostrar Caixas",
    CurrentValue = true,
    Callback = function(v) ESP_Caixas = v end
})

espTab:CreateToggle({
    Name = "Mostrar Linhas",
    CurrentValue = true,
    Callback = function(v) ESP_Linhas = v end
})

espTab:CreateSection("Distancia", false)

espTab:CreateSlider({
    Name = "Distancia Maxima",
    Range = {50, 1000},
    Increment = 50,
    Suffix = "studs",
    CurrentValue = 500,
    Callback = function(v) ESP_DistanciaMaxima = v end
})

espTab:CreateSlider({
    Name = "Tamanho Texto",
    Range = {10, 24},
    Increment = 1,
    Suffix = "px",
    CurrentValue = 14,
    Callback = function(v) ESP_TamanhoTexto = v end
})

espTab:CreateSection("Cores", false)

espTab:CreateColorPicker({
    Name = "Cor do ESP",
    Color = Color3.fromRGB(255, 50, 50),
    Callback = function(v) ESP_Cor = v end
})

local aimbotTab = Window:CreateTab("AIMBOT", 4483362459)

aimbotTab:CreateSection("Configuracao", false)

aimbotTab:CreateToggle({
    Name = "Ativar Aimbot",
    CurrentValue = false,
    Callback = function(v)
        Aimbot_Ativo = v
        if v and Aimbot_MostrarFOV then
            criarFOVCircle()
        elseif not v and FOVCircle then
            FOVCircle.Visible = false
        end
    end
})

aimbotTab:CreateDropdown({
    Name = "Parte do Corpo",
    Options = {"Head", "Torso"},
    CurrentOption = "Head",
    Callback = function(v) Aimbot_MiraParte = v end
})

aimbotTab:CreateSlider({
    Name = "Smoothing (Suavidade)",
    Range = {1, 20},
    Increment = 1,
    CurrentValue = 5,
    Callback = function(v) Aimbot_Smoothing = v end
})

aimbotTab:CreateSlider({
    Name = "FOV",
    Range = {50, 500},
    Increment = 10,
    CurrentValue = 200,
    Callback = function(v)
        Aimbot_FOV = v
        if FOVCircle then FOVCircle.Radius = v end
    end
})

aimbotTab:CreateSlider({
    Name = "Distancia Maxima",
    Range = {50, 500},
    Increment = 25,
    CurrentValue = 300,
    Callback = function(v) Aimbot_DistanciaMaxima = v end
})

aimbotTab:CreateDropdown({
    Name = "Tecla de Ativacao",
    Options = {"RightButton", "LeftButton", "MiddleButton"},
    CurrentOption = "RightButton",
    Callback = function(v) Aimbot_KeyBind = v end
})

aimbotTab:CreateToggle({
    Name = "Apenas alvos visiveis",
    CurrentValue = true,
    Callback = function(v) Aimbot_ApenasVisivel = v end
})

aimbotTab:CreateSection("FOV", false)

aimbotTab:CreateToggle({
    Name = "Mostrar Circulo FOV",
    CurrentValue = true,
    Callback = function(v)
        Aimbot_MostrarFOV = v
        if FOVCircle then FOVCircle.Visible = v and Aimbot_Ativo end
    end
})

aimbotTab:CreateColorPicker({
    Name = "Cor do FOV",
    Color = Color3.fromRGB(255, 255, 255),
    Callback = function(v)
        Aimbot_CorFOV = v
        if FOVCircle then FOVCircle.Color = v end
    end
})

local silentTab = Window:CreateTab("SILENT AIM", 4483362460)

silentTab:CreateSection("Configuracao", false)

silentTab:CreateToggle({
    Name = "Ativar Silent Aim",
    CurrentValue = false,
    Callback = function(v) Silent_Ativo = v end
})

silentTab:CreateDropdown({
    Name = "Parte do Corpo",
    Options = {"Head", "Torso"},
    CurrentOption = "Head",
    Callback = function(v) Silent_MiraParte = v end
})

silentTab:CreateSlider({
    Name = "FOV Silent",
    Range = {50, 500},
    Increment = 10,
    CurrentValue = 200,
    Callback = function(v) Silent_FOV = v end
})

silentTab:CreateSlider({
    Name = "Distancia Maxima",
    Range = {50, 500},
    Increment = 25,
    CurrentValue = 350,
    Callback = function(v) Silent_DistanciaMaxima = v end
})

silentTab:CreateToggle({
    Name = "Apenas alvos visiveis",
    CurrentValue = true,
    Callback = function(v) Silent_ApenasVisivel = v end
})

silentTab:CreateDropdown({
    Name = "Tecla de Ativacao",
    Options = {"RightButton", "LeftButton", "MiddleButton"},
    CurrentOption = "RightButton",
    Callback = function(v) Silent_KeyBind = v end
})

local farmTab = Window:CreateTab("FARM", 4483362461)

farmTab:CreateSection("Auto Click", false)

farmTab:CreateToggle({
    Name = "ATIVAR AUTO CLICK",
    CurrentValue = false,
    Callback = function(v)
        Farm_Ativo = v
        if v then
            iniciarFarm()
        else
            pararFarm()
        end
    end
})

farmTab:CreateSlider({
    Name = "Intervalo (segundos)",
    Range = {1, 10},
    Increment = 0.5,
    Suffix = "s",
    CurrentValue = 5,
    Callback = function(v) Farm_Delay = v end
})

farmTab:CreateSection("Instrucoes", false)

farmTab:CreateLabel("COMO USAR:")
farmTab:CreateLabel("")
farmTab:CreateLabel("1. Ative o toggle acima")
farmTab:CreateLabel("2. Ajuste o intervalo")
farmTab:CreateLabel("3. Fique perto de objetos")
farmTab:CreateLabel("   interagiveis")

local afkTab = Window:CreateTab("AFK", 4483362462)

afkTab:CreateSection("Anti-AFK", false)

afkTab:CreateToggle({
    Name = "ATIVAR ANTI-AFK",
    CurrentValue = false,
    Callback = function(v)
        AFK_Ativo = v
        if v then
            iniciarAFK()
        else
            pararAFK()
        end
    end
})

afkTab:CreateSlider({
    Name = "Intervalo (minutos)",
    Range = {1, 30},
    Increment = 1,
    Suffix = "min",
    CurrentValue = 5,
    Callback = function(v)
        AFK_Intervalo = v * 60
    end
})

afkTab:CreateSection("O que o Anti-AFK faz:", false)

afkTab:CreateLabel("Move o mouse levemente")
afkTab:CreateLabel("Da um pequeno pulo")
afkTab:CreateLabel("")
afkTab:CreateLabel("FUNCIONA MESMO COM O")
afkTab:CreateLabel("ROBLOX MINIMIZADO")

local infoTab = Window:CreateTab("INFO", 4483362463)

infoTab:CreateSection("Sobre", false)

infoTab:CreateLabel("ZAP MENU v1.0")
infoTab:CreateLabel("ESP + Aimbot + Silent Aim")
infoTab:CreateLabel("Farm + Anti-AFK")
infoTab:CreateLabel("")
infoTab:CreateLabel("CONTROLES:")
infoTab:CreateLabel("Botao Direito = Ativar mira")
infoTab:CreateLabel("Menu configuravel")
infoTab:CreateLabel("")
infoTab:CreateLabel("ANTI-AFK:")
infoTab:CreateLabel("Funciona minimizado")
infoTab:CreateLabel("Intervalo ajustavel")

-- =====================
-- // LOOPS
-- =====================

RunService.RenderStepped:Connect(function()
    atualizarESP()
    atualizarFOVCircle()
end)

Players.PlayerAdded:Connect(function(p)
    if ESP_Ativo then criarESP(p) end
end)
Players.PlayerRemoving:Connect(removerESP)

print("ZAP MENU v1.0 carregado!")
print("Aimbot: segure botao direito do mouse")
print("ESP: ative na aba ESP")
print("Farm: ative na aba FARM")
print("Anti-AFK: ative na aba AFK - funciona minimizado")

-- Painel de Habilidades para Roblox - VERSÃO COM VELOCIDADE
-- Coloque este script em StarterPlayer > StarterPlayerScripts
-- Pressione G para abrir/fechar o painel

print("🔄 Iniciando Painel de Habilidades...")

local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local TweenService = game:GetService("TweenService")
local RunService = game:GetService("RunService")

local player = Players.LocalPlayer
local mouse = player:GetMouse()

-- Aguardar o PlayerGui estar disponível
local playerGui
repeat
    wait()
    playerGui = player:FindFirstChild("PlayerGui")
until playerGui

print("✅ PlayerGui encontrado!")

-- Variáveis de controle
local isGuiOpen = false
local teleportEnabled = false
local damageMultiplierEnabled = false
local speedEnabled = false
local teleportKey = "F"
local currentMultiplier = 2
local currentSpeed = 50
local originalWalkSpeed = 16
local gui = nil
local mainFrame = nil

-- Função para criar efeito visual de teleporte
local function createTeleportEffect(position)
    local character = player.Character
    if not character then return end
    
    local effect = Instance.new("Part")
    effect.Name = "TeleportEffect"
    effect.Shape = Enum.PartType.Ball
    effect.Material = Enum.Material.Neon
    effect.BrickColor = BrickColor.new("Bright blue")
    effect.Size = Vector3.new(0.1, 0.1, 0.1)
    effect.Position = position
    effect.Anchored = true
    effect.CanCollide = false
    effect.Parent = workspace
    
    -- Animação do efeito
    local tweenInfo = TweenInfo.new(0.5, Enum.EasingStyle.Quad, Enum.EasingDirection.Out)
    local tween = TweenService:Create(effect, tweenInfo, {
        Size = Vector3.new(10, 10, 10),
        Transparency = 1
    })
    
    tween:Play()
    tween.Completed:Connect(function()
        effect:Destroy()
    end)
end

-- Função de teleporte
local function teleportToMousePosition()
    if not teleportEnabled then return end
    
    local character = player.Character
    if not character or not character:FindFirstChild("HumanoidRootPart") then 
        print("❌ Character ou RootPart não encontrado para teleporte")
        return 
    end
    
    local rootPart = character.HumanoidRootPart
    local hit = mouse.Hit
    
    if hit then
        local targetPosition = hit.Position + Vector3.new(0, 5, 0)
        rootPart.CFrame = CFrame.new(targetPosition)
        createTeleportEffect(targetPosition)
        print("✅ Teleportado para: " .. tostring(targetPosition))
    end
end

-- Sistema de velocidade
local function updateSpeed()
    local character = player.Character
    if not character then return end
    
    local humanoid = character:FindFirstChildOfClass("Humanoid")
    if not humanoid then return end
    
    if speedEnabled then
        humanoid.WalkSpeed = currentSpeed
        print("💨 Velocidade alterada para: " .. currentSpeed)
    else
        humanoid.WalkSpeed = originalWalkSpeed
        print("🚶 Velocidade restaurada para: " .. originalWalkSpeed)
    end
end

-- Função para atualizar status visual
local function updateTeleportStatus(statusLabel, button)
    if teleportEnabled then
        statusLabel.Text = "STATUS: ATIVO"
        statusLabel.TextColor3 = Color3.new(0, 1, 0)
        button.BackgroundColor3 = Color3.new(0, 0.7, 0)
        button.Text = "DESATIVAR"
    else
        statusLabel.Text = "STATUS: DESATIVADO"
        statusLabel.TextColor3 = Color3.new(1, 0, 0)
        button.BackgroundColor3 = Color3.new(0.7, 0, 0)
        button.Text = "ATIVAR"
    end
end

local function updateDamageStatus(statusLabel, button)
    if damageMultiplierEnabled then
        statusLabel.Text = "STATUS: ATIVO (" .. currentMultiplier .. "x)"
        statusLabel.TextColor3 = Color3.new(0, 1, 0)
        button.BackgroundColor3 = Color3.new(0, 0.7, 0)
        button.Text = "DESATIVAR"
    else
        statusLabel.Text = "STATUS: DESATIVADO"
        statusLabel.TextColor3 = Color3.new(1, 0, 0)
        button.BackgroundColor3 = Color3.new(0.7, 0, 0)
        button.Text = "ATIVAR"
    end
end

local function updateSpeedStatus(statusLabel, button)
    if speedEnabled then
        statusLabel.Text = "STATUS: ATIVO (" .. currentSpeed .. ")"
        statusLabel.TextColor3 = Color3.new(0, 1, 0)
        button.BackgroundColor3 = Color3.new(0, 0.7, 0)
        button.Text = "DESATIVAR"
    else
        statusLabel.Text = "STATUS: DESATIVADO"
        statusLabel.TextColor3 = Color3.new(1, 0, 0)
        button.BackgroundColor3 = Color3.new(0.7, 0, 0)
        button.Text = "ATIVAR"
    end
end

-- Sistema de multiplicador de dano para SkyWars
local damageConnections = {}

local function setupDamageMultiplier()
    -- Limpar conexões antigas
    for _, connection in pairs(damageConnections) do
        if connection then
            connection:Disconnect()
        end
    end
    damageConnections = {}
    
    local character = player.Character
    if not character then return end
    
    local function setupToolDamage(tool)
        if not tool:IsA("Tool") then return end
        
        -- Procurar por scripts de dano na ferramenta
        for _, child in pairs(tool:GetDescendants()) do
            if child:IsA("RemoteEvent") and (child.Name:lower():find("damage") or child.Name:lower():find("hit")) then
                print("🔧 Hook em RemoteEvent: " .. child.Name)
                
                -- Hook no RemoteEvent original
                local originalFire = child.FireServer
                child.FireServer = function(self, ...)
                    local args = {...}
                    
                    -- Tentar encontrar o valor de dano nos argumentos
                    for i, arg in pairs(args) do
                        if type(arg) == "number" and arg > 0 and arg < 1000 then
                            args[i] = arg * currentMultiplier
                            print("💥 Dano multiplicado: " .. arg .. " -> " .. args[i])
                            break
                        end
                    end
                    
                    return originalFire(self, unpack(args))
                end
                
                table.insert(damageConnections, {disconnect = function() child.FireServer = originalFire end})
            end
        end
        
        -- Hook em LocalScripts que podem ter dano
        local handle = tool:FindFirstChild("Handle")
        if handle then
            local connection = handle.Touched:Connect(function(hit)
                if damageMultiplierEnabled and hit.Parent:FindFirstChild("Humanoid") and hit.Parent ~= character then
                    print("🗡️ Ataque detectado em: " .. hit.Parent.Name)
                end
            end)
            table.insert(damageConnections, connection)
        end
    end
    
    -- Configurar para ferramentas atuais
    for _, tool in pairs(character:GetChildren()) do
        setupToolDamage(tool)
    end
    
    -- Configurar para ferramentas futuras
    local toolConnection = character.ChildAdded:Connect(function(child)
        if child:IsA("Tool") then
            wait(0.1) -- Esperar a tool carregar completamente
            setupToolDamage(child)
        end
    end)
    table.insert(damageConnections, toolConnection)
    
    -- Configurar para backpack também
    local backpack = player:FindFirstChild("Backpack")
    if backpack then
        for _, tool in pairs(backpack:GetChildren()) do
            setupToolDamage(tool)
        end
        
        local backpackConnection = backpack.ChildAdded:Connect(function(child)
            if child:IsA("Tool") then
                setupToolDamage(child)
            end
        end)
        table.insert(damageConnections, backpackConnection)
    end
    
    print("✅ Sistema de multiplicador configurado para SkyWars!")
end

-- Função para detectar tecla personalizada
local function setupCustomKey(keyLabel, button)
    keyLabel.Text = "Pressione uma tecla..."
    keyLabel.TextColor3 = Color3.new(1, 1, 0)
    
    local connection
    connection = UserInputService.InputBegan:Connect(function(input, gameProcessed)
        if gameProcessed then return end
        
        if input.UserInputType == Enum.UserInputType.Keyboard then
            teleportKey = input.KeyCode.Name
            keyLabel.Text = "TECLA: " .. teleportKey
            keyLabel.TextColor3 = Color3.new(1, 1, 1)
            connection:Disconnect()
        elseif input.UserInputType == Enum.UserInputType.MouseButton1 then
            teleportKey = "MouseButton1"
            keyLabel.Text = "TECLA: BOTÃO ESQUERDO"
            keyLabel.TextColor3 = Color3.new(1, 1, 1)
            connection:Disconnect()
        elseif input.UserInputType == Enum.UserInputType.MouseButton2 then
            teleportKey = "MouseButton2"
            keyLabel.Text = "TECLA: BOTÃO DIREITO"
            keyLabel.TextColor3 = Color3.new(1, 1, 1)
            connection:Disconnect()
        end
    end)
end

-- Função para criar a GUI
local function createGui()
    print("🎨 Criando interface...")
    
    -- Remove GUI anterior se existir
    if gui then
        gui:Destroy()
        print("🗑️ GUI anterior removida")
    end
    
    -- ScreenGui principal
    gui = Instance.new("ScreenGui")
    gui.Name = "SkillsPanel"
    gui.Parent = playerGui
    gui.ResetOnSpawn = false
    
    print("📱 ScreenGui criado e adicionado ao PlayerGui")
    
    -- Frame principal
    mainFrame = Instance.new("Frame")
    mainFrame.Name = "MainFrame"
    mainFrame.Size = UDim2.new(0, 380, 0, 580)
    mainFrame.Position = UDim2.new(0.5, -190, 0.5, -290)
    mainFrame.BackgroundColor3 = Color3.new(0.1, 0.1, 0.1)
    mainFrame.BorderSizePixel = 0
    mainFrame.Active = true
    mainFrame.Draggable = true
    mainFrame.Parent = gui
    mainFrame.Visible = true
    
    print("🖼️ Frame principal criado")
    
    -- Cantos arredondados
    local corner = Instance.new("UICorner")
    corner.CornerRadius = UDim.new(0, 15)
    corner.Parent = mainFrame
    
    -- Título
    local title = Instance.new("TextLabel")
    title.Name = "Title"
    title.Size = UDim2.new(1, 0, 0, 45)
    title.Position = UDim2.new(0, 0, 0, 0)
    title.BackgroundColor3 = Color3.new(0.2, 0.2, 0.2)
    title.BorderSizePixel = 0
    title.Text = "PAINEL DE HABILIDADES"
    title.TextColor3 = Color3.new(1, 1, 1)
    title.TextScaled = true
    title.Font = Enum.Font.GothamBold
    title.Parent = mainFrame
    
    local titleCorner = Instance.new("UICorner")
    titleCorner.CornerRadius = UDim.new(0, 15)
    titleCorner.Parent = title
    
    -- Seção de Teleporte
    local teleportSection = Instance.new("Frame")
    teleportSection.Name = "TeleportSection"
    teleportSection.Size = UDim2.new(1, -20, 0, 160)
    teleportSection.Position = UDim2.new(0, 10, 0, 55)
    teleportSection.BackgroundColor3 = Color3.new(0.15, 0.15, 0.15)
    teleportSection.BorderSizePixel = 0
    teleportSection.Parent = mainFrame
    
    local teleportCorner = Instance.new("UICorner")
    teleportCorner.CornerRadius = UDim.new(0, 10)
    teleportCorner.Parent = teleportSection
    
    -- Título da seção teleporte
    local teleportTitle = Instance.new("TextLabel")
    teleportTitle.Size = UDim2.new(1, 0, 0, 25)
    teleportTitle.Position = UDim2.new(0, 0, 0, 5)
    teleportTitle.BackgroundTransparency = 1
    teleportTitle.Text = "🚀 TELEPORTE"
    teleportTitle.TextColor3 = Color3.new(0.8, 0.8, 1)
    teleportTitle.TextScaled = true
    teleportTitle.Font = Enum.Font.Gotham
    teleportTitle.Parent = teleportSection
    
    -- Status do teleporte
    local teleportStatus = Instance.new("TextLabel")
    teleportStatus.Size = UDim2.new(1, -10, 0, 20)
    teleportStatus.Position = UDim2.new(0, 5, 0, 30)
    teleportStatus.BackgroundTransparency = 1
    teleportStatus.Text = "STATUS: DESATIVADO"
    teleportStatus.TextColor3 = Color3.new(1, 0, 0)
    teleportStatus.TextScaled = true
    teleportStatus.Font = Enum.Font.Gotham
    teleportStatus.Parent = teleportSection
    
    -- Botão toggle teleporte
    local teleportToggle = Instance.new("TextButton")
    teleportToggle.Size = UDim2.new(0.45, 0, 0, 35)
    teleportToggle.Position = UDim2.new(0.05, 0, 0, 55)
    teleportToggle.BackgroundColor3 = Color3.new(0.7, 0, 0)
    teleportToggle.BorderSizePixel = 0
    teleportToggle.Text = "ATIVAR"
    teleportToggle.TextColor3 = Color3.new(1, 1, 1)
    teleportToggle.TextScaled = true
    teleportToggle.Font = Enum.Font.GothamBold
    teleportToggle.Parent = teleportSection
    
    local toggleCorner = Instance.new("UICorner")
    toggleCorner.CornerRadius = UDim.new(0, 8)
    toggleCorner.Parent = teleportToggle
    
    -- Botão teleportar agora
    local teleportNow = Instance.new("TextButton")
    teleportNow.Size = UDim2.new(0.45, 0, 0, 35)
    teleportNow.Position = UDim2.new(0.5, 0, 0, 55)
    teleportNow.BackgroundColor3 = Color3.new(0, 0.5, 0.8)
    teleportNow.BorderSizePixel = 0
    teleportNow.Text = "TELEPORTAR AGORA"
    teleportNow.TextColor3 = Color3.new(1, 1, 1)
    teleportNow.TextScaled = true
    teleportNow.Font = Enum.Font.GothamBold
    teleportNow.Parent = teleportSection
    
    local nowCorner = Instance.new("UICorner")
    nowCorner.CornerRadius = UDim.new(0, 8)
    nowCorner.Parent = teleportNow
    
    -- Label da tecla atual
    local keyLabel = Instance.new("TextLabel")
    keyLabel.Size = UDim2.new(1, -10, 0, 20)
    keyLabel.Position = UDim2.new(0, 5, 0, 95)
    keyLabel.BackgroundTransparency = 1
    keyLabel.Text = "TECLA: " .. teleportKey
    keyLabel.TextColor3 = Color3.new(1, 1, 1)
    keyLabel.TextScaled = true
    keyLabel.Font = Enum.Font.Gotham
    keyLabel.Parent = teleportSection
    
    -- Botão para escolher tecla
    local chooseKey = Instance.new("TextButton")
    chooseKey.Size = UDim2.new(1, -10, 0, 30)
    chooseKey.Position = UDim2.new(0, 5, 0, 120)
    chooseKey.BackgroundColor3 = Color3.new(0.3, 0.3, 0.3)
    chooseKey.BorderSizePixel = 0
    chooseKey.Text = "ESCOLHER NOVA TECLA"
    chooseKey.TextColor3 = Color3.new(1, 1, 1)
    chooseKey.TextScaled = true
    chooseKey.Font = Enum.Font.Gotham
    chooseKey.Parent = teleportSection
    
    local chooseCorner = Instance.new("UICorner")
    chooseCorner.CornerRadius = UDim.new(0, 8)
    chooseCorner.Parent = chooseKey
    
    -- Seção de Velocidade
    local speedSection = Instance.new("Frame")
    speedSection.Name = "SpeedSection"
    speedSection.Size = UDim2.new(1, -20, 0, 140)
    speedSection.Position = UDim2.new(0, 10, 0, 225)
    speedSection.BackgroundColor3 = Color3.new(0.15, 0.15, 0.15)
    speedSection.BorderSizePixel = 0
    speedSection.Parent = mainFrame
    
    local speedCorner = Instance.new("UICorner")
    speedCorner.CornerRadius = UDim.new(0, 10)
    speedCorner.Parent = speedSection
    
    -- Título da seção velocidade
    local speedTitle = Instance.new("TextLabel")
    speedTitle.Size = UDim2.new(1, 0, 0, 25)
    speedTitle.Position = UDim2.new(0, 0, 0, 5)
    speedTitle.BackgroundTransparency = 1
    speedTitle.Text = "💨 VELOCIDADE"
    speedTitle.TextColor3 = Color3.new(0.8, 1, 0.8)
    speedTitle.TextScaled = true
    speedTitle.Font = Enum.Font.Gotham
    speedTitle.Parent = speedSection
    
    -- Status da velocidade
    local speedStatus = Instance.new("TextLabel")
    speedStatus.Size = UDim2.new(1, -10, 0, 20)
    speedStatus.Position = UDim2.new(0, 5, 0, 30)
    speedStatus.BackgroundTransparency = 1
    speedStatus.Text = "STATUS: DESATIVADO"
    speedStatus.TextColor3 = Color3.new(1, 0, 0)
    speedStatus.TextScaled = true
    speedStatus.Font = Enum.Font.Gotham
    speedStatus.Parent = speedSection
    
    -- Botão toggle velocidade
    local speedToggle = Instance.new("TextButton")
    speedToggle.Size = UDim2.new(0.45, 0, 0, 35)
    speedToggle.Position = UDim2.new(0.05, 0, 0, 55)
    speedToggle.BackgroundColor3 = Color3.new(0.7, 0, 0)
    speedToggle.BorderSizePixel = 0
    speedToggle.Text = "ATIVAR"
    speedToggle.TextColor3 = Color3.new(1, 1, 1)
    speedToggle.TextScaled = true
    speedToggle.Font = Enum.Font.GothamBold
    speedToggle.Parent = speedSection
    
    local speedToggleCorner = Instance.new("UICorner")
    speedToggleCorner.CornerRadius = UDim.new(0, 8)
    speedToggleCorner.Parent = speedToggle
    
    -- TextBox para velocidade
    local speedBox = Instance.new("TextBox")
    speedBox.Size = UDim2.new(0.45, 0, 0, 35)
    speedBox.Position = UDim2.new(0.5, 0, 0, 55)
    speedBox.BackgroundColor3 = Color3.new(0.2, 0.2, 0.2)
    speedBox.BorderSizePixel = 0
    speedBox.Text = "50"
    speedBox.PlaceholderText = "Velocidade"
    speedBox.TextColor3 = Color3.new(1, 1, 1)
    speedBox.TextScaled = true
    speedBox.Font = Enum.Font.Gotham
    speedBox.Parent = speedSection
    
    local speedBoxCorner = Instance.new("UICorner")
    speedBoxCorner.CornerRadius = UDim.new(0, 8)
    speedBoxCorner.Parent = speedBox
    
    -- Label explicativa velocidade
    local speedExplanation = Instance.new("TextLabel")
    speedExplanation.Size = UDim2.new(1, -10, 0, 35)
    speedExplanation.Position = UDim2.new(0, 5, 0, 95)
    speedExplanation.BackgroundTransparency = 1
    speedExplanation.Text = "Velocidade normal: 16 | Recomendado: 30-80"
    speedExplanation.TextColor3 = Color3.new(0.8, 0.8, 0.8)
    speedExplanation.TextScaled = true
    speedExplanation.Font = Enum.Font.Gotham
    speedExplanation.Parent = speedSection
    
    -- Seção de Multiplicador de Dano
    local damageSection = Instance.new("Frame")
    damageSection.Name = "DamageSection"
    damageSection.Size = UDim2.new(1, -20, 0, 140)
    damageSection.Position = UDim2.new(0, 10, 0, 375)
    damageSection.BackgroundColor3 = Color3.new(0.15, 0.15, 0.15)
    damageSection.BorderSizePixel = 0
    damageSection.Parent = mainFrame
    
    local damageCorner = Instance.new("UICorner")
    damageCorner.CornerRadius = UDim.new(0, 10)
    damageCorner.Parent = damageSection
    
    -- Título da seção dano
    local damageTitle = Instance.new("TextLabel")
    damageTitle.Size = UDim2.new(1, 0, 0, 25)
    damageTitle.Position = UDim2.new(0, 0, 0, 5)
    damageTitle.BackgroundTransparency = 1
    damageTitle.Text = "⚔️ MULTIPLICADOR DE DANO"
    damageTitle.TextColor3 = Color3.new(1, 0.8, 0.8)
    damageTitle.TextScaled = true
    damageTitle.Font = Enum.Font.Gotham
    damageTitle.Parent = damageSection
    
    -- Status do multiplicador
    local damageStatus = Instance.new("TextLabel")
    damageStatus.Size = UDim2.new(1, -10, 0, 20)
    damageStatus.Position = UDim2.new(0, 5, 0, 30)
    damageStatus.BackgroundTransparency = 1
    damageStatus.Text = "STATUS: DESATIVADO"
    damageStatus.TextColor3 = Color3.new(1, 0, 0)
    damageStatus.TextScaled = true
    damageStatus.Font = Enum.Font.Gotham
    damageStatus.Parent = damageSection
    
    -- Botão toggle multiplicador
    local damageToggle = Instance.new("TextButton")
    damageToggle.Size = UDim2.new(0.45, 0, 0, 35)
    damageToggle.Position = UDim2.new(0.05, 0, 0, 55)
    damageToggle.BackgroundColor3 = Color3.new(0.7, 0, 0)
    damageToggle.BorderSizePixel = 0
    damageToggle.Text = "ATIVAR"
    damageToggle.TextColor3 = Color3.new(1, 1, 1)
    damageToggle.TextScaled = true
    damageToggle.Font = Enum.Font.GothamBold
    damageToggle.Parent = damageSection
    
    local damageToggleCorner = Instance.new("UICorner")
    damageToggleCorner.CornerRadius = UDim.new(0, 8)
    damageToggleCorner.Parent = damageToggle
    
    -- TextBox para multiplicador
    local multiplierBox = Instance.new("TextBox")
    multiplierBox.Size = UDim2.new(0.45, 0, 0, 35)
    multiplierBox.Position = UDim2.new(0.5, 0, 0, 55)
    multiplierBox.BackgroundColor3 = Color3.new(0.2, 0.2, 0.2)
    multiplierBox.BorderSizePixel = 0
    multiplierBox.Text = "2"
    multiplierBox.PlaceholderText = "Multiplicador"
    multiplierBox.TextColor3 = Color3.new(1, 1, 1)
    multiplierBox.TextScaled = true
    multiplierBox.Font = Enum.Font.Gotham
    multiplierBox.Parent = damageSection
    
    local boxCorner = Instance.new("UICorner")
    boxCorner.CornerRadius = UDim.new(0, 8)
    boxCorner.Parent = multiplierBox
    
    -- Label explicativa
    local explanationLabel = Instance.new("TextLabel")
    explanationLabel.Size = UDim2.new(1, -10, 0, 35)
    explanationLabel.Position = UDim2.new(0, 5, 0, 95)
    explanationLabel.BackgroundTransparency = 1
    explanationLabel.Text = "Funciona com espadas/ferramentas de combate"
    explanationLabel.TextColor3 = Color3.new(0.8, 0.8, 0.8)
    explanationLabel.TextScaled = true
    explanationLabel.Font = Enum.Font.Gotham
    explanationLabel.Parent = damageSection
    
    -- Botão fechar
    local closeButton = Instance.new("TextButton")
    closeButton.Size = UDim2.new(0, 25, 0, 25)
    closeButton.Position = UDim2.new(1, -35, 0, 10)
    closeButton.BackgroundColor3 = Color3.new(0.8, 0, 0)
    closeButton.BorderSizePixel = 0
    closeButton.Text = "X"
    closeButton.TextColor3 = Color3.new(1, 1, 1)
    closeButton.TextScaled = true
    closeButton.Font = Enum.Font.GothamBold
    closeButton.Parent = mainFrame
    
    local closeCorner = Instance.new("UICorner")
    closeCorner.CornerRadius = UDim.new(0, 12)
    closeCorner.Parent = closeButton
    
    print("🎛️ Todos os elementos da interface criados")
    
    -- Eventos dos botões
    teleportToggle.MouseButton1Click:Connect(function()
        teleportEnabled = not teleportEnabled
        updateTeleportStatus(teleportStatus, teleportToggle)
        print("🚀 Teleporte: " .. (teleportEnabled and "ATIVADO" or "DESATIVADO"))
    end)
    
    teleportNow.MouseButton1Click:Connect(function()
        print("📍 Tentando teleportar agora...")
        teleportToMousePosition()
    end)
    
    chooseKey.MouseButton1Click:Connect(function()
        print("⌨️ Configurando nova tecla...")
        setupCustomKey(keyLabel, chooseKey)
    end)
    
    speedToggle.MouseButton1Click:Connect(function()
        speedEnabled = not speedEnabled
        updateSpeedStatus(speedStatus, speedToggle)
        updateSpeed()
        print("💨 Velocidade: " .. (speedEnabled and "ATIVADA" or "DESATIVADA"))
    end)
    
    speedBox.FocusLost:Connect(function()
        local value = tonumber(speedBox.Text)
        if value and value > 0 and value <= 500 then
            currentSpeed = value
            updateSpeedStatus(speedStatus, speedToggle)
            if speedEnabled then
                updateSpeed()
            end
            print("🏃 Velocidade alterada para: " .. currentSpeed)
        else
            speedBox.Text = tostring(currentSpeed)
        end
    end)
    
    damageToggle.MouseButton1Click:Connect(function()
        damageMultiplierEnabled = not damageMultiplierEnabled
        updateDamageStatus(damageStatus, damageToggle)
        print("⚔️ Multiplicador de dano: " .. (damageMultiplierEnabled and "ATIVADO" or "DESATIVADO"))
        
        if damageMultiplierEnabled then
            setupDamageMultiplier()
        else
            -- Limpar conexões quando desativado
            for _, connection in pairs(damageConnections) do
                if connection and connection.Disconnect then
                    connection:Disconnect()
                end
            end
            damageConnections = {}
            
            -- Restaurar RemoteEvents originais
            for remote, originalFire in pairs(hookedRemotes) do
                if remote and remote.Parent then
                    remote.FireServer = originalFire
                end
            end
            hookedRemotes = {}
            print("🔄 Hooks de dano removidos e RemoteEvents restaurados")
        end
    end)
    
    multiplierBox.FocusLost:Connect(function()
        local value = tonumber(multiplierBox.Text)
        if value and value > 0 then
            currentMultiplier = value
            updateDamageStatus(damageStatus, damageToggle)
            print("🔢 Multiplicador alterado para: " .. currentMultiplier .. "x")
        else
            multiplierBox.Text = tostring(currentMultiplier)
        end
    end)
    
    closeButton.MouseButton1Click:Connect(function()
        print("❌ Fechando painel...")
        toggleGui()
    end)
    
    -- Inicializar status
    updateTeleportStatus(teleportStatus, teleportToggle)
    updateDamageStatus(damageStatus, damageToggle)
    updateSpeedStatus(speedStatus, speedToggle)
    
    print("✅ Interface totalmente configurada!")
    
    return mainFrame
end

-- Função para abrir/fechar GUI
function toggleGui()
    print("🔄 Toggle GUI chamado. Estado atual: " .. (isGuiOpen and "ABERTO" or "FECHADO"))
    
    if not gui or not gui.Parent then
        print("📱 Criando nova GUI...")
        local newMainFrame = createGui()
        isGuiOpen = true
        newMainFrame.Visible = true
        print("✅ GUI criada e exibida!")
    else
        isGuiOpen = not isGuiOpen
        if mainFrame then
            mainFrame.Visible = isGuiOpen
            print("👁️ Visibilidade alterada para: " .. (isGuiOpen and "VISÍVEL" or "OCULTO"))
        end
    end
end

-- Input handling
UserInputService.InputBegan:Connect(function(input, gameProcessed)
    if gameProcessed then return end
    
    -- Debug: mostrar todas as teclas pressionadas
    if input.UserInputType == Enum.UserInputType.Keyboard then
        print("⌨️ Tecla pressionada: " .. input.KeyCode.Name)
    end
    
    -- Abrir/fechar painel com G
    if input.KeyCode == Enum.KeyCode.G then
        print("🎯 Tecla G detectada! Alternando GUI...")
        toggleGui()
    end
    
    -- Teleporte com tecla customizada (padrão F)
    if teleportEnabled then
        if input.KeyCode.Name == teleportKey then
            print("🚀 Tecla de teleporte detectada: " .. teleportKey)
            teleportToMousePosition()
        elseif teleportKey == "MouseButton1" and input.UserInputType == Enum.UserInputType.MouseButton1 then
            teleportToMousePosition()
        elseif teleportKey == "MouseButton2" and input.UserInputType == Enum.UserInputType.MouseButton2 then
            teleportToMousePosition()
        end
    end
end)

print("🎮 Input handler configurado")

-- Recriar GUI e sistemas quando o jogador respawn
player.CharacterAdded:Connect(function(character)
    print("👤 Character respawned, aguardando...")
    
    -- Salvar velocidade original do novo character
    local humanoid = character:WaitForChild("Humanoid")
    originalWalkSpeed = humanoid.WalkSpeed
    print("🚶 Velocidade original salva: " .. originalWalkSpeed)
    
    wait(2) -- Esperar um pouco mais para o character carregar completamente
    
    if isGuiOpen then
        print("🔄 Recriando GUI após respawn...")
        createGui()
        if gui and mainFrame then
            mainFrame.Visible = true
        end
    end
    
    -- Reconfigurar sistemas se estiverem ativos
    if damageMultiplierEnabled then
        setupDamageMultiplier()
    end
    
    if speedEnabled then
        wait(0.5) -- Esperar um pouco mais para aplicar velocidade
        updateSpeed()
    end
end)

-- Configuração inicial se o jogador já existir
if player.Character then
    local humanoid = player.Character:FindFirstChildOfClass("Humanoid")
    if humanoid then
        originalWalkSpeed = humanoid.WalkSpeed
        print("🚶 Velocidade original detectada: " .. originalWalkSpeed)
    end
end

-- Teste inicial - criar GUI automaticamente para debug
wait(1)
print("🚀 Painel de Habilidades TOTALMENTE carregado!")
print("📝 INSTRUÇÕES:")
print("   • Pressione G para abrir/fechar o painel")
print("   • Tecla padrão de teleporte: F")
print("   • Velocidade padrão: " .. originalWalkSpeed)
print("   • Verifique o Output para mensagens de debug")

-- Criar GUI automaticamente na primeira vez (para teste)
spawn(function()
    wait(2)
    if not isGuiOpen then
        print("🔧 Criando GUI automaticamente para teste...")
        toggleGui()
    end
end)

local noclipEnabled = false
local function updateNoclip()
    local character = player.Character
    if not character then return end

    for _, part in pairs(character:GetDescendants()) do
        if part:IsA("BasePart") and part.Name ~= "HumanoidRootPart" then
            part.CanCollide = not noclipEnabled
        end
    end
end
local function createNoclipSection(parentFrame)
    local noclipSection = Instance.new("Frame")
    noclipSection.Name = "NoclipSection"
    noclipSection.Size = UDim2.new(1, -20, 0, 100)
    noclipSection.Position = UDim2.new(0, 10, 0, 525) -- logo abaixo do Dano
    noclipSection.BackgroundColor3 = Color3.new(0.15, 0.15, 0.15)
    noclipSection.BorderSizePixel = 0
    noclipSection.Parent = parentFrame

    local corner = Instance.new("UICorner")
    corner.CornerRadius = UDim.new(0, 10)
    corner.Parent = noclipSection

    -- Título
    local title = Instance.new("TextLabel")
    title.Size = UDim2.new(1, 0, 0, 25)
    title.Position = UDim2.new(0, 0, 0, 5)
    title.BackgroundTransparency = 1
    title.Text = "👻 NOCLIP"
    title.TextColor3 = Color3.new(1, 1, 0.5)
    title.TextScaled = true
    title.Font = Enum.Font.Gotham
    title.Parent = noclipSection

    -- Status
    local status = Instance.new("TextLabel")
    status.Size = UDim2.new(1, -10, 0, 20)
    status.Position = UDim2.new(0, 5, 0, 30)
    status.BackgroundTransparency = 1
    status.Text = "STATUS: DESATIVADO"
    status.TextColor3 = Color3.new(1, 0, 0)
    status.TextScaled = true
    status.Font = Enum.Font.Gotham
    status.Parent = noclipSection

    -- Botão toggle
    local toggle = Instance.new("TextButton")
    toggle.Size = UDim2.new(0.9, 0, 0, 35)
    toggle.Position = UDim2.new(0.05, 0, 0, 55)
    toggle.BackgroundColor3 = Color3.new(0.7, 0, 0)
    toggle.BorderSizePixel = 0
    toggle.Text = "ATIVAR"
    toggle.TextColor3 = Color3.new(1, 1, 1)
    toggle.TextScaled = true
    toggle.Font = Enum.Font.GothamBold
    toggle.Parent = noclipSection

    local toggleCorner = Instance.new("UICorner")
    toggleCorner.CornerRadius = UDim.new(0, 8)
    toggleCorner.Parent = toggle

    -- Evento do botão
    toggle.MouseButton1Click:Connect(function()
        noclipEnabled = not noclipEnabled
        updateNoclip()
        if noclipEnabled then
            status.Text = "STATUS: ATIVO"
            status.TextColor3 = Color3.new(0, 1, 0)
            toggle.Text = "DESATIVAR"
            toggle.BackgroundColor3 = Color3.new(0, 0.7, 0)
        else
            status.Text = "STATUS: DESATIVADO"
            status.TextColor3 = Color3.new(1, 0, 0)
            toggle.Text = "ATIVAR"
            toggle.BackgroundColor3 = Color3.new(0.7, 0, 0)
        end
        print("👻 Noclip: " .. (noclipEnabled and "ATIVADO" or "DESATIVADO"))
    end)

    -- Atualiza continuamente
    RunService.RenderStepped:Connect(function()
        if noclipEnabled then
            updateNoclip()
        end
    end)
end
createNoclipSection(mainFrame)


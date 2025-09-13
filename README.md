-- Painel de Habilidades para Roblox - VERSÃO COM VELOCIDADE E NOCLIP
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
local noclipEnabled = false
local teleportKey = "F"
local currentMultiplier = 2
local currentSpeed = 50
local originalWalkSpeed = 16
local gui = nil
local mainFrame = nil
local damageConnections = {}
local hookedRemotes = {}

-- Função de teleporte
local function createTeleportEffect(position)
    local character = player.Character
    if not character then return end
    
    local effect = Instance.new("Part")
    effect.Name = "TeleportEffect"
    effect.Shape = Enum.PartType.Ball
    effect.Material = Enum.Material.Neon
    effect.BrickColor = BrickColor.new("Bright blue")
    effect.Size = Vector3.new(0.1,0.1,0.1)
    effect.Position = position
    effect.Anchored = true
    effect.CanCollide = false
    effect.Parent = workspace
    
    local tweenInfo = TweenInfo.new(0.5, Enum.EasingStyle.Quad, Enum.EasingDirection.Out)
    local tween = TweenService:Create(effect, tweenInfo, {
        Size = Vector3.new(10,10,10),
        Transparency = 1
    })
    tween:Play()
    tween.Completed:Connect(function()
        effect:Destroy()
    end)
end

local function teleportToMousePosition()
    if not teleportEnabled then return end
    local character = player.Character
    if not character or not character:FindFirstChild("HumanoidRootPart") then return end
    local rootPart = character.HumanoidRootPart
    local hit = mouse.Hit
    if hit then
        local targetPosition = hit.Position + Vector3.new(0,5,0)
        rootPart.CFrame = CFrame.new(targetPosition)
        createTeleportEffect(targetPosition)
        print("✅ Teleportado para: " .. tostring(targetPosition))
    end
end

-- Função de velocidade
local function updateSpeed()
    local character = player.Character
    if not character then return end
    local humanoid = character:FindFirstChildOfClass("Humanoid")
    if not humanoid then return end
    if speedEnabled then
        humanoid.WalkSpeed = currentSpeed
    else
        humanoid.WalkSpeed = originalWalkSpeed
    end
end

-- Função de Noclip
local function updateNoclip()
    local character = player.Character
    if not character then return end
    for _, part in pairs(character:GetDescendants()) do
        if part:IsA("BasePart") and part.Name ~= "HumanoidRootPart" then
            part.CanCollide = not noclipEnabled
        end
    end
end

RunService.RenderStepped:Connect(function()
    if noclipEnabled then
        updateNoclip()
    end
end)

-- Funções de status visual
local function updateTeleportStatus(statusLabel, button)
    if teleportEnabled then
        statusLabel.Text = "STATUS: ATIVO"
        statusLabel.TextColor3 = Color3.new(0,1,0)
        button.BackgroundColor3 = Color3.new(0,0.7,0)
        button.Text = "DESATIVAR"
    else
        statusLabel.Text = "STATUS: DESATIVADO"
        statusLabel.TextColor3 = Color3.new(1,0,0)
        button.BackgroundColor3 = Color3.new(0.7,0,0)
        button.Text = "ATIVAR"
    end
end

local function updateDamageStatus(statusLabel, button)
    if damageMultiplierEnabled then
        statusLabel.Text = "STATUS: ATIVO (" .. currentMultiplier .. "x)"
        statusLabel.TextColor3 = Color3.new(0,1,0)
        button.BackgroundColor3 = Color3.new(0,0.7,0)
        button.Text = "DESATIVAR"
    else
        statusLabel.Text = "STATUS: DESATIVADO"
        statusLabel.TextColor3 = Color3.new(1,0,0)
        button.BackgroundColor3 = Color3.new(0.7,0,0)
        button.Text = "ATIVAR"
    end
end

local function updateSpeedStatus(statusLabel, button)
    if speedEnabled then
        statusLabel.Text = "STATUS: ATIVO (" .. currentSpeed .. ")"
        statusLabel.TextColor3 = Color3.new(0,1,0)
        button.BackgroundColor3 = Color3.new(0,0.7,0)
        button.Text = "DESATIVAR"
    else
        statusLabel.Text = "STATUS: DESATIVADO"
        statusLabel.TextColor3 = Color3.new(1,0,0)
        button.BackgroundColor3 = Color3.new(0.7,0,0)
        button.Text = "ATIVAR"
    end
end

-- Função para configurar key customizada
local function setupCustomKey(keyLabel, button)
    keyLabel.Text = "Pressione uma tecla..."
    keyLabel.TextColor3 = Color3.new(1,1,0)
    local connection
    connection = UserInputService.InputBegan:Connect(function(input, gameProcessed)
        if gameProcessed then return end
        if input.UserInputType == Enum.UserInputType.Keyboard then
            teleportKey = input.KeyCode.Name
            keyLabel.Text = "TECLA: " .. teleportKey
            keyLabel.TextColor3 = Color3.new(1,1,1)
            connection:Disconnect()
        elseif input.UserInputType == Enum.UserInputType.MouseButton1 then
            teleportKey = "MouseButton1"
            keyLabel.Text = "TECLA: BOTÃO ESQUERDO"
            keyLabel.TextColor3 = Color3.new(1,1,1)
            connection:Disconnect()
        elseif input.UserInputType == Enum.UserInputType.MouseButton2 then
            teleportKey = "MouseButton2"
            keyLabel.Text = "TECLA: BOTÃO DIREITO"
            keyLabel.TextColor3 = Color3.new(1,1,1)
            connection:Disconnect()
        end
    end)
end

-- Função de Multiplicador de Dano
local function setupDamageMultiplier()
    for _, connection in pairs(damageConnections) do
        if connection then connection:Disconnect() end
    end
    damageConnections = {}

    local character = player.Character
    if not character then return end

    local function setupToolDamage(tool)
        if not tool:IsA("Tool") then return end
        for _, child in pairs(tool:GetDescendants()) do
            if child:IsA("RemoteEvent") and (child.Name:lower():find("damage") or child.Name:lower():find("hit")) then
                local originalFire = child.FireServer
                child.FireServer = function(self,...)
                    local args = {...}
                    for i,arg in pairs(args) do
                        if type(arg)=="number" and arg>0 and arg<1000 then
                            args[i] = arg*currentMultiplier
                            break
                        end
                    end
                    return originalFire(self, unpack(args))
                end
                hookedRemotes[child] = originalFire
            end
        end
    end

    for _, tool in pairs(character:GetChildren()) do
        setupToolDamage(tool)
    end

    local toolConnection = character.ChildAdded:Connect(function(child)
        if child:IsA("Tool") then wait(0.1) setupToolDamage(child) end
    end)
    table.insert(damageConnections, toolConnection)

    local backpack = player:FindFirstChild("Backpack")
    if backpack then
        for _, tool in pairs(backpack:GetChildren()) do
            setupToolDamage(tool)
        end
        local backpackConnection = backpack.ChildAdded:Connect(function(child)
            if child:IsA("Tool") then setupToolDamage(child) end
        end)
        table.insert(damageConnections, backpackConnection)
    end

    print("✅ Sistema de multiplicador configurado!")
end

-- Função GUI
local function createGui()
    if gui then gui:Destroy() end
    gui = Instance.new("ScreenGui")
    gui.Name = "SkillsPanel"
    gui.Parent = playerGui
    gui.ResetOnSpawn = false

    mainFrame = Instance.new("Frame")
    mainFrame.Name = "MainFrame"
    mainFrame.Size = UDim2.new(0,380,0,650)
    mainFrame.Position = UDim2.new(0.5,-190,0.5,-325)
    mainFrame.BackgroundColor3 = Color3.new(0.1,0.1,0.1)
    mainFrame.BorderSizePixel = 0
    mainFrame.Active = true
    mainFrame.Draggable = true
    mainFrame.Parent = gui
    mainFrame.Visible = true

    local corner = Instance.new("UICorner")
    corner.CornerRadius = UDim.new(0,15)
    corner.Parent = mainFrame

    local title = Instance.new("TextLabel")
    title.Size = UDim2.new(1,0,0,45)
    title.Position = UDim2.new(0,0,0,0)
    title.BackgroundColor3 = Color3.new(0.2,0.2,0.2)
    title.BorderSizePixel = 0
    title.Text = "PAINEL DE HABILIDADES"
    title.TextColor3 = Color3.new(1,1,1)
    title.TextScaled = true
    title.Font = Enum.Font.GothamBold
    title.Parent = mainFrame
    local titleCorner = Instance.new("UICorner")
    titleCorner.CornerRadius = UDim.new(0,15)
    titleCorner.Parent = title

    -- [INSERIR SEÇÕES TELEPORTE, VELOCIDADE, DANO AQUI]...
    -- Use o código do seu script original para criar essas seções normalmente

    -- ======== Seção Noclip =========
    local noclipSection = Instance.new("Frame")
    noclipSection.Name = "NoclipSection"
    noclipSection.Size = UDim2.new(1,-20,0,100)
    noclipSection.Position = UDim2.new(0,10,0,525)
    noclipSection.BackgroundColor3 = Color3.new(0.15,0.15,0.15)
    noclipSection.BorderSizePixel = 0
    noclipSection.Parent = mainFrame
    local cornerN = Instance.new("UICorner")
    cornerN.CornerRadius = UDim.new(0,10)
    cornerN.Parent = noclipSection

    local titleN = Instance.new("TextLabel")
    titleN.Size = UDim2.new(1,0,0,25)
    titleN.Position = UDim2.new(0,0,0,5)
    titleN.BackgroundTransparency = 1
    titleN.Text = "👻 NOCLIP"
    titleN.TextColor3 = Color3.new(1,1,0.5)
    titleN.TextScaled = true
    titleN.Font = Enum.Font.Gotham
    titleN.Parent = noclipSection

    local statusN = Instance.new("TextLabel")
    statusN.Size = UDim2.new(1,-10,0,20)
    statusN.Position = UDim2.new(0,5,0,30)
    statusN.BackgroundTransparency = 1
    statusN.Text = "STATUS: DESATIVADO"
    statusN.TextColor3 = Color3.new(1,0,0)
    statusN.TextScaled = true
    statusN.Font = Enum.Font.Gotham
    statusN.Parent = noclipSection

    local toggleN = Instance.new("TextButton")
    toggleN.Size = UDim2.new(0.9,0,0,35)
    toggleN.Position = UDim2.new(0.05,0,0,55)
    toggleN.BackgroundColor3 = Color3.new(0.7,0,0)
    toggleN.BorderSizePixel = 0
    toggleN.Text = "ATIVAR"
    toggleN.TextColor3 = Color3.new(1,1,1)
    toggleN.TextScaled = true
    toggleN.Font = Enum.Font.GothamBold
    toggleN.Parent = noclipSection

    local toggleCorner = Instance.new("UICorner")
    toggleCorner.CornerRadius = UDim.new(0,8)
    toggleCorner.Parent = toggleN

    toggleN.MouseButton1Click:Connect(function()
        noclipEnabled = not noclipEnabled
        statusN.Text = noclipEnabled and "STATUS: ATIVO" or "STATUS: DESATIVADO"
        statusN.TextColor3 = noclipEnabled and Color3.new(0,1,0) or Color3.new(1,0,0)
        toggleN.Text = noclipEnabled and "DESATIVAR" or "ATIVAR"
        toggleN.BackgroundColor3 = noclipEnabled and Color3.new(0,0.7,0) or Color3.new(0.7,0,0)
        updateNoclip()
    end)
end

-- Abrir GUI
createGui()

-- Toggle GUI com G
UserInputService.InputBegan:Connect(function(input, gameProcessed)
    if gameProcessed then return end
    if input.KeyCode == Enum.KeyCode.G then
        isGuiOpen = not isGuiOpen
        if mainFrame then mainFrame.Visible = isGuiOpen end
    end
end)

-- Teleporte key
UserInputService.InputBegan:Connect(function(input, gameProcessed)
    if gameProcessed then return end
    if teleportEnabled and input.KeyCode.Name == teleportKey then
        teleportToMousePosition()
    end
end)

-- Atualiza velocidade continuamente
RunService.RenderStepped:Connect(function()
    updateSpeed()
end)

-- Detecta respawn
player.CharacterAdded:Connect(function()
    wait(1)
    updateSpeed()
    updateNoclip()
    setupDamageMultiplier()
end)

print("✅ Painel de Habilidades carregado com sucesso!")

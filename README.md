-- Painel de Habilidades para Roblox - VERSÃO COM VELOCIDADE + Noclip
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
local speedEnabled = false
local noclipEnabled = false
local teleportKey = "F"
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
    if not character or not character:FindFirstChild("HumanoidRootPart") then return end
    
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

-- Sistema de noclip
local function toggleNoclip(state)
    noclipEnabled = state
    local character = player.Character
    if not character then return end
    
    for _, part in pairs(character:GetDescendants()) do
        if part:IsA("BasePart") and part.Name ~= "HumanoidRootPart" then
            part.CanCollide = not noclipEnabled
        end
    end
end

-- Função para detectar tecla personalizada
local function setupCustomKey(keyLabel)
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

-- Função para criar GUI
local function createGui()
    if gui then gui:Destroy() end
    
    gui = Instance.new("ScreenGui")
    gui.Name = "SkillsPanel"
    gui.Parent = playerGui
    gui.ResetOnSpawn = false
    
    mainFrame = Instance.new("Frame")
    mainFrame.Size = UDim2.new(0, 380, 0, 480)
    mainFrame.Position = UDim2.new(0.5, -190, 0.5, -240)
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
    
    -- Seção Teleporte
    local teleportSection = Instance.new("Frame")
    teleportSection.Size = UDim2.new(1,-20,0,120)
    teleportSection.Position = UDim2.new(0,10,0,55)
    teleportSection.BackgroundColor3 = Color3.new(0.15,0.15,0.15)
    teleportSection.BorderSizePixel = 0
    teleportSection.Parent = mainFrame
    
    local teleportCorner = Instance.new("UICorner")
    teleportCorner.CornerRadius = UDim.new(0,10)
    teleportCorner.Parent = teleportSection
    
    local teleportStatus = Instance.new("TextLabel")
    teleportStatus.Size = UDim2.new(1,-10,0,20)
    teleportStatus.Position = UDim2.new(0,5,0,5)
    teleportStatus.BackgroundTransparency = 1
    teleportStatus.Text = "STATUS: DESATIVADO"
    teleportStatus.TextColor3 = Color3.new(1,0,0)
    teleportStatus.TextScaled = true
    teleportStatus.Font = Enum.Font.Gotham
    teleportStatus.Parent = teleportSection
    
    local teleportToggle = Instance.new("TextButton")
    teleportToggle.Size = UDim2.new(0.45,0,0,35)
    teleportToggle.Position = UDim2.new(0.05,0,0,30)
    teleportToggle.BackgroundColor3 = Color3.new(0.7,0,0)
    teleportToggle.BorderSizePixel = 0
    teleportToggle.Text = "ATIVAR"
    teleportToggle.TextColor3 = Color3.new(1,1,1)
    teleportToggle.TextScaled = true
    teleportToggle.Font = Enum.Font.GothamBold
    teleportToggle.Parent = teleportSection
    
    local keyLabel = Instance.new("TextLabel")
    keyLabel.Size = UDim2.new(1,-10,0,20)
    keyLabel.Position = UDim2.new(0,5,0,70)
    keyLabel.BackgroundTransparency = 1
    keyLabel.Text = "TECLA: "..teleportKey
    keyLabel.TextColor3 = Color3.new(1,1,1)
    keyLabel.TextScaled = true
    keyLabel.Font = Enum.Font.Gotham
    keyLabel.Parent = teleportSection
    
    local chooseKey = Instance.new("TextButton")
    chooseKey.Size = UDim2.new(1,-10,0,25)
    chooseKey.Position = UDim2.new(0,5,0,95)
    chooseKey.BackgroundColor3 = Color3.new(0.3,0.3,0.3)
    chooseKey.BorderSizePixel = 0
    chooseKey.Text = "ESCOLHER NOVA TECLA"
    chooseKey.TextColor3 = Color3.new(1,1,1)
    chooseKey.TextScaled = true
    chooseKey.Font = Enum.Font.Gotham
    chooseKey.Parent = teleportSection
    
    -- Seção Velocidade
    local speedSection = Instance.new("Frame")
    speedSection.Size = UDim2.new(1,-20,0,100)
    speedSection.Position = UDim2.new(0,10,0,190)
    speedSection.BackgroundColor3 = Color3.new(0.15,0.15,0.15)
    speedSection.BorderSizePixel = 0
    speedSection.Parent = mainFrame
    
    local speedStatus = Instance.new("TextLabel")
    speedStatus.Size = UDim2.new(1,-10,0,20)
    speedStatus.Position = UDim2.new(0,5,0,5)
    speedStatus.BackgroundTransparency = 1
    speedStatus.Text = "STATUS: DESATIVADO"
    speedStatus.TextColor3 = Color3.new(1,0,0)
    speedStatus.TextScaled = true
    speedStatus.Font = Enum.Font.Gotham
    speedStatus.Parent = speedSection
    
    local speedToggle = Instance.new("TextButton")
    speedToggle.Size = UDim2.new(0.45,0,0,35)
    speedToggle.Position = UDim2.new(0.05,0,0,30)
    speedToggle.BackgroundColor3 = Color3.new(0.7,0,0)
    speedToggle.BorderSizePixel = 0
    speedToggle.Text = "ATIVAR"
    speedToggle.TextColor3 = Color3.new(1,1,1)
    speedToggle.TextScaled = true
    speedToggle.Font = Enum.Font.GothamBold
    speedToggle.Parent = speedSection
    
    local speedBox = Instance.new("TextBox")
    speedBox.Size = UDim2.new(0.45,0,0,35)
    speedBox.Position = UDim2.new(0.5,0,0,30)
    speedBox.BackgroundColor3 = Color3.new(0.2,0.2,0.2)
    speedBox.BorderSizePixel = 0
    speedBox.Text = tostring(currentSpeed)
    speedBox.PlaceholderText = "Velocidade"
    speedBox.TextColor3 = Color3.new(1,1,1)
    speedBox.TextScaled = true
    speedBox.Font = Enum.Font.Gotham
    speedBox.Parent = speedSection
    
    -- Seção Noclip
    local noclipSection = Instance.new("Frame")
    noclipSection.Size = UDim2.new(1,-20,0,60)
    noclipSection.Position = UDim2.new(0,10,0,310)
    noclipSection.BackgroundColor3 = Color3.new(0.15,0.15,0.15)
    noclipSection.BorderSizePixel = 0
    noclipSection.Parent = mainFrame
    
    local noclipStatus = Instance.new("TextLabel")
    noclipStatus.Size = UDim2.new(1,-10,0,20)
    noclipStatus.Position = UDim2.new(0,5,0,5)
    noclipStatus.BackgroundTransparency = 1
    noclipStatus.Text = "Noclip: DESATIVADO"
    noclipStatus.TextColor3 = Color3.new(1,0,0)
    noclipStatus.TextScaled = true
    noclipStatus.Font = Enum.Font.Gotham
    noclipStatus.Parent = noclipSection
    
    local noclipToggle = Instance.new("TextButton")
    noclipToggle.Size = UDim2.new(1,-10,0,35)
    noclipToggle.Position = UDim2.new(0,5,0,25)
    noclipToggle.BackgroundColor3 = Color3.new(0.7,0,0)
    noclipToggle.BorderSizePixel = 0
    noclipToggle.Text = "ATIVAR"
    noclipToggle.TextColor3 = Color3.new(1,1,1)
    noclipToggle.TextScaled = true
    noclipToggle.Font = Enum.Font.GothamBold
    noclipToggle.Parent = noclipSection
    
    -- Botões funcionalidade
    teleportToggle.MouseButton1Click:Connect(function()
        teleportEnabled = not teleportEnabled
        teleportStatus.Text = teleportEnabled and "STATUS: ATIVO" or "STATUS: DESATIVADO"
        teleportStatus.TextColor3 = teleportEnabled and Color3.new(0,1,0) or Color3.new(1,0,0)
    end)
    
    chooseKey.MouseButton1Click:Connect(function()
        setupCustomKey(keyLabel)
    end)
    
    speedToggle.MouseButton1Click:Connect(function()
        speedEnabled = not speedEnabled
        speedStatus.Text = speedEnabled and "STATUS: ATIVO" or "STATUS: DESATIVADO"
        speedStatus.TextColor3 = speedEnabled and Color3.new(0,1,0) or Color3.new(1,0,0)
        updateSpeed()
    end)
    
    speedBox.FocusLost:Connect(function()
        local value = tonumber(speedBox.Text)
        if value and value > 0 and value <= 500 then
            currentSpeed = value
            if speedEnabled then updateSpeed() end
        else
            speedBox.Text = tostring(currentSpeed)
        end
    end)
    
    noclipToggle.MouseButton1Click:Connect(function()
        noclipEnabled = not noclipEnabled
        noclipStatus.Text = noclipEnabled and "Noclip: ATIVADO" or "Noclip: DESATIVADO"
        noclipStatus.TextColor3 = noclipEnabled and Color3.new(0,1,0) or Color3.new(1,0,0)
        toggleNoclip(noclipEnabled)
    end)
    
    return mainFrame
end

-- Função toggle GUI
function toggleGui()
    if not gui or not gui.Parent then
        createGui()
        isGuiOpen = true
    else
        isGuiOpen = not isGuiOpen
        if mainFrame then mainFrame.Visible = isGuiOpen end
    end
end

-- Input handling
UserInputService.InputBegan:Connect(function(input, gameProcessed)
    if gameProcessed then return end
    
    if input.KeyCode == Enum.KeyCode.G then
        toggleGui()
    end
    
    if teleportEnabled then
        if input.KeyCode.Name == teleportKey then
            teleportToMousePosition()
        elseif teleportKey == "MouseButton1" and input.UserInputType == Enum.UserInputType.MouseButton1 then
            teleportToMousePosition()
        elseif teleportKey == "MouseButton2" and input.UserInputType == Enum.UserInputType.MouseButton2 then
            teleportToMousePosition()
        end
    end
end)

-- Reaplicar velocidade e noclip após respawn
player.CharacterAdded:Connect(function(character)
    local humanoid = character:WaitForChild("Humanoid")
    originalWalkSpeed = humanoid.WalkSpeed
    wait(1)
    if speedEnabled then updateSpeed() end
    if noclipEnabled then toggleNoclip(true) end
end)

print("🎮 Painel de Habilidades carregado com Noclip!")

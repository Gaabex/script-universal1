-- Serviços
local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local RunService = game:GetService("RunService")
local LocalPlayer = Players.LocalPlayer
local Mouse = LocalPlayer:GetMouse()

local Character = LocalPlayer.Character or LocalPlayer.CharacterAdded:Wait()
local Humanoid = Character:WaitForChild("Humanoid")
local RootPart = Character:WaitForChild("HumanoidRootPart")

-- Criando a GUI
local ScreenGui = Instance.new("ScreenGui", LocalPlayer:WaitForChild("PlayerGui"))
ScreenGui.Name = "PainelHabilidades"

local MainFrame = Instance.new("Frame")
MainFrame.Size = UDim2.new(0, 350, 0, 500)
MainFrame.Position = UDim2.new(0.65, 0, 0.2, 0)
MainFrame.BackgroundColor3 = Color3.fromRGB(30,30,30)
MainFrame.BorderSizePixel = 0
MainFrame.Visible = false -- começa oculto
MainFrame.Parent = ScreenGui

local UIListLayout = Instance.new("UIListLayout")
UIListLayout.Parent = MainFrame
UIListLayout.Padding = UDim.new(0, 5)

-- Função auxiliar: criar botão
local function criarBotao(nome, funcao, valorInicial)
    local botao = Instance.new("TextButton")
    botao.Size = UDim2.new(1, -10, 0, 40)
    botao.BackgroundColor3 = Color3.fromRGB(150,0,0)
    botao.TextColor3 = Color3.new(1,1,1)
    botao.Text = nome.." [DESLIGADO]"
    botao.Parent = MainFrame

    local ativo = false
    local valor = valorInicial or nil

    local function atualizarVisual()
        if ativo then
            botao.BackgroundColor3 = Color3.fromRGB(0,150,0)
            botao.Text = nome.." [LIGADO]"
        else
            botao.BackgroundColor3 = Color3.fromRGB(150,0,0)
            botao.Text = nome.." [DESLIGADO]"
        end
    end

    botao.MouseButton1Click:Connect(function()
        ativo = not ativo
        atualizarVisual()
        funcao(ativo, valor)
    end)

    return {
        botao = botao,
        setValue = function(v) valor = v end,
        getValue = function() return valor end,
        isActive = function() return ativo end
    }
end

-- Função auxiliar: criar slider (valor numérico)
local function criarSlider(nome, valorMin, valorMax, valorInicial)
    local frame = Instance.new("Frame")
    frame.Size = UDim2.new(1, -10, 0, 40)
    frame.BackgroundColor3 = Color3.fromRGB(50,50,50)
    frame.Parent = MainFrame

    local label = Instance.new("TextLabel")
    label.Size = UDim2.new(0.5, 0, 1, 0)
    label.Position = UDim2.new(0, 0, 0, 0)
    label.BackgroundTransparency = 1
    label.TextColor3 = Color3.new(1,1,1)
    label.Text = nome..": "..valorInicial
    label.Font = Enum.Font.SourceSansBold
    label.TextScaled = true
    label.Parent = frame

    local textbox = Instance.new("TextBox")
    textbox.Size = UDim2.new(0.5, -5, 1, -5)
    textbox.Position = UDim2.new(0.5, 5, 0, 0)
    textbox.Text = tostring(valorInicial)
    textbox.ClearTextOnFocus = false
    textbox.TextColor3 = Color3.new(1,1,1)
    textbox.BackgroundColor3 = Color3.fromRGB(80,80,80)
    textbox.Parent = frame

    local valor = valorInicial

    textbox.FocusLost:Connect(function()
        local n = tonumber(textbox.Text)
        if n and n >= valorMin and n <= valorMax then
            valor = n
            label.Text = nome..": "..valor
        else
            textbox.Text = tostring(valor)
        end
    end)

    return {
        frame = frame,
        getValue = function() return valor end
    }
end

-- Tabela de habilidades
local habilidades = {}

-- Velocidade
local velocidadeSlider = criarSlider("Velocidade", 16, 200, 50)
habilidades["Velocidade"] = criarBotao("Velocidade", function(ativo, valor)
    if ativo then
        Humanoid.WalkSpeed = velocidadeSlider.getValue()
    else
        Humanoid.WalkSpeed = 16
    end
end, velocidadeSlider.getValue())

-- Atualizar velocidade continuamente se ativo
RunService.RenderStepped:Connect(function()
    if habilidades["Velocidade"].isActive() then
        Humanoid.WalkSpeed = velocidadeSlider.getValue()
    end
end)

-- Pulo Alto
local puloSlider = criarSlider("Pulo", 50, 300, 100)
habilidades["Pulo Alto"] = criarBotao("Pulo Alto", function(ativo, valor)
    if ativo then
        Humanoid.JumpPower = puloSlider.getValue()
    else
        Humanoid.JumpPower = 50
    end
end, puloSlider.getValue())

-- Atualizar pulo continuamente
RunService.RenderStepped:Connect(function()
    if habilidades["Pulo Alto"].isActive() then
        Humanoid.JumpPower = puloSlider.getValue()
    end
end)

-- Noclip
habilidades["Noclip"] = criarBotao("Noclip", function(ativo)
    if ativo then
        for _, part in pairs(Character:GetDescendants()) do
            if part:IsA("BasePart") then
                part.CanCollide = false
            end
        end
    else
        for _, part in pairs(Character:GetDescendants()) do
            if part:IsA("BasePart") then
                part.CanCollide = true
            end
        end
    end
end)

-- Teleporte
local teleKey = Enum.KeyCode.T -- padrão
habilidades["Teleporte"] = criarBotao("Teleporte", function() end)

UIS.InputBegan:Connect(function(input, gpe)
    if gpe then return end
    if input.KeyCode == teleKey and habilidades["Teleporte"].isActive() then
        local pos = Mouse.Hit.Position + Vector3.new(0,3,0)
        Character:SetPrimaryPartCFrame(CFrame.new(pos))
    end
end)

-- AutoAim
habilidades["AutoAim"] = criarBotao("AutoAim", function(ativo)
    print("AutoAim "..(ativo and "ativado" or "desativado"))
end)

-- KillAura
habilidades["KillAura"] = criarBotao("KillAura", function(ativo)
    print("KillAura "..(ativo and "ativado" or "desativado"))
end)

-- Multiplicador de Dano
habilidades["Multiplicador"] = criarBotao("Multiplicador de Dano", function(ativo)
    print("Multiplicador "..(ativo and "ativado" or "desativado"))
end)

-- Regeneração Rápida
habilidades["Regeneração"] = criarBotao("Regeneração Rápida", function(ativo)
    if ativo then
        spawn(function()
            while habilidades["Regeneração"].isActive() do
                Humanoid.Health = math.min(Humanoid.MaxHealth, Humanoid.Health + 5)
                wait(0.5)
            end
        end)
    end
end)

-- Knockback
local knockbackSlider = criarSlider("Knockback", 10, 500, 50)
habilidades["Knockback"] = criarBotao("Knockback", function(ativo)
    print("Knockback "..(ativo and "ativado" or "desativado"))
end)

-- Mostrar/Ocultar painel com G
local toggleKey = Enum.KeyCode.G
UserInputService.InputBegan:Connect(function(input, gpe)
    if gpe then return end
    if input.KeyCode == toggleKey then
        MainFrame.Visible = not MainFrame.Visible
    end
end)

--// SERVIÇOS DO SISTEMA
local TweenService = game:GetService("TweenService")
local UserInputService = game:GetService("UserInputService")
local RunService = game:GetService("RunService")
local Players = game:GetService("Players")

local Player = Players.LocalPlayer
local Character = Player.Character or Player.CharacterAdded:Wait()
local Humanoid = Character:WaitForChild("Humanoid")
local RootPart = Character:WaitForChild("HumanoidRootPart")

--// CONFIGURAÇÕES E ESTADOS
local COLORS = {
    Background = Color3.fromRGB(2, 10, 25), -- Azul escuro profundo
    Secondary = Color3.fromRGB(5, 25, 55), -- Azul escuro médio
    Accent = Color3.fromRGB(0, 200, 255),  -- Ciano brilhante neon
    Text = Color3.fromRGB(255, 255, 255),   -- Branco purpúreo
    Glow = Color3.fromRGB(0, 160, 255)     -- Ciano de brilho suave
}

local States = {
    Desync = false,
    InfJump = false, -- Novo Estado para Infinite Jump
    NoAnim = false,
    Visible = true
}

local CloneModel = nil -- Variável para armazenar o Clone do Desync
local infJumpInputConnection = nil -- Conexão para o evento InputBegan

--// INTERFACE PRINCIPAL
local screenGui = Instance.new("ScreenGui")
screenGui.Name = "HarbixDesyncUI"
screenGui.ResetOnSpawn = false
screenGui.Parent = Player:WaitForChild("PlayerGui")

local mainFrame = Instance.new("Frame")
mainFrame.Name = "MainFrame"
mainFrame.Size = UDim2.new(0, 280, 0, 250) -- Aumentado ligeiramente para 4 opções
mainFrame.Position = UDim2.new(0.5, -140, 0.4, 0)
mainFrame.BackgroundColor3 = COLORS.Background
mainFrame.BorderSizePixel = 0
mainFrame.Parent = screenGui

Instance.new("UICorner", mainFrame).CornerRadius = UDim.new(0, 15)

-- Brilho Neon (Stroke)
local mainStroke = Instance.new("UIStroke", mainFrame)
mainStroke.Color = COLORS.Accent
mainStroke.Thickness = 2.5
mainStroke.ApplyStrokeMode = Enum.ApplyStrokeMode.Border

-- Título Estilizado (⚡ HARBIX DESYNC)
local title = Instance.new("TextLabel")
title.Text = "⚡ HARBIX\n   DESYNC"
title.Size = UDim2.new(0, 150, 0, 50)
title.Position = UDim2.new(0, 15, 0, 10)
title.BackgroundTransparency = 1
title.TextColor3 = COLORS.Accent
title.Font = Enum.Font.SpecialElite
title.TextSize = 18
title.LineHeight = 0.8
title.TextXAlignment = Enum.TextXAlignment.Left
title.Parent = mainFrame

-- Botão Fechar (✕)
local closeBtn = Instance.new("TextButton")
closeBtn.Text = "✕"
closeBtn.Size = UDim2.new(0, 28, 0, 28)
closeBtn.Position = UDim2.new(1, -38, 0, 12)
closeBtn.BackgroundColor3 = COLORS.Secondary
closeBtn.TextColor3 = COLORS.Accent
closeBtn.Font = Enum.Font.GothamBold
closeBtn.Parent = mainFrame
Instance.new("UICorner", closeBtn).CornerRadius = UDim.new(0, 8)

--// LÓGICA DAS FUNÇÕES

-- 1. Desync de Clone (🌀 - Toggle)
local function ToggleDesync(enable)
    local char = Player.Character
    if not char or not char:FindFirstChild("HumanoidRootPart") then return end

    if enable then
        -- Criar o Clone Visual
        char.Archivable = true
        CloneModel = char:Clone()
        CloneModel.Parent = workspace
        CloneModel:SetPrimaryPartCFrame(char.HumanoidRootPart.CFrame)
        
        -- Deixar o clone fantasma e inofensivo
        for _, part in pairs(CloneModel:GetDescendants()) do
            if part:IsA("BasePart") then
                part.Transparency = 0.3
                part.CanCollide = false
            elseif part:IsA("Script") or part:IsA("LocalScript") or part:IsA("TouchTransmitter") then
                part:Destroy() -- Previne detecção e bugs
            end
        end
        
        -- "Mover" o jogador real para longe (simulação de desync)
        char.HumanoidRootPart.CFrame = char.HumanoidRootPart.CFrame * CFrame.new(0, 500, 0)
    else
        -- Voltar ao normal
        if CloneModel and CloneModel:FindFirstChild("HumanoidRootPart") then
            char.HumanoidRootPart.CFrame = CloneModel.HumanoidRootPart.CFrame
            CloneModel:Destroy()
            CloneModel = nil
        end
    end
end

-- 2. Infinite Jump (🚀 - Toggle - STABLE METHOD)
-- Esta abordagem usa Velocity direta no RootPart ao invés de JumpRequest.
-- É mais imune a bugs de morte do nada causados pela física de queda.
local function ToggleInfJump(enable)
    if enable then
        -- Conectar o evento de entrada de teclado
        infJumpInputConnection = UserInputService.InputBegan:Connect(function(input, proc)
            -- Se não estiver digitando no chat e pressionar Espaço
            if not proc and input.KeyCode == Enum.KeyCode.Space then
                if States.InfJump and Character and Humanoid and Humanoid.Health > 0 and RootPart then
                    -- Aplica velocidade vertical instantânea baseada no JumpPower do personagem
                    -- Isso força o pulo sem alterar o estado de queda interno (prevenindo bugs)
                    RootPart.Velocity = Vector3.new(RootPart.Velocity.X, Humanoid.JumpPower, RootPart.Velocity.Z)
                end
            end
        end)
    else
        -- Desconectar o evento
        if infJumpInputConnection then
            infJumpInputConnection:Disconnect()
            infJumpInputConnection = nil
        end
    end
end

-- 3. Insta Reset (💀 - Tecla G / Botão)
local function PerformInstaReset()
    if Player.Character and Humanoid then
        Humanoid.Health = 0
    end
end

-- Monitoramento da tecla G para Reset
UserInputService.InputBegan:Connect(function(input, proc)
    if not proc and input.KeyCode == Enum.KeyCode.G then
        PerformInstaReset()
    end
end)

-- 4. No Anim (🚫 - Toggle)
RunService.RenderStepped:Connect(function()
    if States.NoAnim and Humanoid then
        for _, track in pairs(Humanoid:GetPlayingAnimationTracks()) do
            track:Stop()
        end
    end
end)

--// SISTEMA DE GERAÇÃO DE INTERFACE (Mesma interface visual)
local function CreateOptionRow(name, emoji, pos, key, stateKey, clickAction)
    local row = Instance.new("TextButton")
    row.Size = UDim2.new(0.92, 0, 0, 40)
    row.Position = pos
    row.BackgroundColor3 = COLORS.Secondary
    row.BackgroundTransparency = 0.4
    row.AutoButtonColor = false
    row.Text = ""
    row.Parent = mainFrame
    Instance.new("UICorner", row).CornerRadius = UDim.new(0, 10)
    
    local rowStroke = Instance.new("UIStroke", row)
    rowStroke.Color = COLORS.Accent
    rowStroke.Transparency = 0.7

    local label = Instance.new("TextLabel")
    label.Text = emoji .. " " .. name
    label.Size = UDim2.new(0.65, 0, 1, 0)
    label.Position = UDim2.new(0, 12, 0, 0)
    label.BackgroundTransparency = 1
    label.TextColor3 = COLORS.Text
    label.Font = Enum.Font.Ubuntu
    label.TextSize = 14
    label.TextXAlignment = Enum.TextXAlignment.Left
    label.Parent = row

    if key then
        local keyLabel = Instance.new("TextLabel")
        keyLabel.Text = "["..key.."]"
        keyLabel.Size = UDim2.new(0, 30, 1, 0)
        keyLabel.Position = UDim2.new(1, -42, 0, 0)
        keyLabel.BackgroundTransparency = 1
        keyLabel.TextColor3 = COLORS.Accent
        keyLabel.Font = Enum.Font.GothamMedium
        keyLabel.TextSize = 13
        keyLabel.TextXAlignment = Enum.TextXAlignment.Center
        keyLabel.Parent = row
    end

    if stateKey then
        local switchBase = Instance.new("Frame")
        switchBase.Size = UDim2.new(0, 38, 0, 18)
        switchBase.Position = UDim2.new(0.82, 0, 0.28, 0)
        switchBase.BackgroundColor3 = Color3.new(0,0,0)
        switchBase.Parent = row
        Instance.new("UICorner", switchBase).CornerRadius = UDim.new(1,0)
        
        local dot = Instance.new("Frame")
        dot.Size = UDim2.new(0, 12, 0, 12)
        dot.Position = UDim2.new(0.1, 0, 0.15, 0)
        dot.BackgroundColor3 = COLORS.Accent
        dot.Parent = switchBase
        Instance.new("UICorner", dot).CornerRadius = UDim.new(1,0)
        
        row.MouseButton1Click:Connect(function()
            States[stateKey] = not States[stateKey]
            
            TweenService:Create(dot, TweenInfo.new(0.25, Enum.EasingStyle.Sine), {
                Position = States[stateKey] and UDim2.new(0.6, 0, 0.15, 0) or UDim2.new(0.1, 0, 0.15, 0)
            }):Play()
            TweenService:Create(switchBase, TweenInfo.new(0.2, Enum.EasingStyle.Sine), {
                BackgroundColor3 = States[stateKey] and COLORS.Accent or Color3.new(0,0,0)
            }):Play()
            
            if clickAction then
                clickAction(States[stateKey])
            end
        end)
    elseif clickAction then
        row.MouseButton1Click:Connect(clickAction)
    end
end

--// CRIAR AS OPÇÕES NA INTERFACE (Mesma disposição visual)
CreateOptionRow("Desync (Clone)", "🌀", UDim2.new(0.04, 0, 0.28, 0), nil, "Desync", ToggleDesync)
CreateOptionRow("Infinite Jump (Stable)", "🚀", UDim2.new(0.04, 0, 0.48, 0), nil, "InfJump", ToggleInfJump)
CreateOptionRow("Insta Reset", "💀", UDim2.new(0.04, 0, 0.68, 0), "G", nil, PerformInstaReset)
CreateOptionRow("No Anim", "🚫", UDim2.new(0.04, 0, 0.88, 0), nil, "NoAnim", nil)

--// DRAGGABLE E ANIMAÇÕES DO MENU

local dragging, dragStart, startPos
mainFrame.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        dragging = true; dragStart = input.Position; startPos = mainFrame.Position
    end
end)
UserInputService.InputChanged:Connect(function(input)
    if dragging and input.UserInputType == Enum.UserInputType.MouseMovement then
        local delta = input.Position - dragStart
        mainFrame.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
    end
end)
UserInputService.InputEnded:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then dragging = false end
end)

-- Loop de Brilho Neon Pulsante (Efeito Visual)
task.spawn(function()
    while true do
        local ti = TweenInfo.new(1.8, Enum.EasingStyle.Sine, Enum.EasingDirection.InOut)
        TweenService:Create(mainStroke, ti, {Transparency = 0.85, Thickness = 4}):Play()
        task.wait(1.8)
        TweenService:Create(mainStroke, ti, {Transparency = 0.1, Thickness = 2.5}):Play()
        task.wait(1.8)
    end
end)

-- Alternar Visibilidade com a Tecla Q
UserInputService.InputBegan:Connect(function(input, proc)
    if not proc and input.KeyCode == Enum.KeyCode.Q then
        States.Visible = not States.Visible
        mainFrame.Visible = States.Visible
    end
end)

closeBtn.MouseButton1Click:Connect(function() screenGui:Destroy() end)

-- Garantir que a referência do Humanoid e Character sejam atualizadas no respawn
Player.CharacterAdded:Connect(function(char)
    Character = char
    Humanoid = char:WaitForChild("Humanoid")
    RootPart = char:WaitForChild("HumanoidRootPart")
    -- Reconecta o pulo infinito se já estava ativado
    if States.InfJump then
        ToggleInfJump(false) -- Remove conexão antiga
        ToggleInfJump(true)  -- Cria nova conexão
    end
end)

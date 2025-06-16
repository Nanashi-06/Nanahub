-- Verificação de compatibilidade de PlaceId
local PlaceIdPermitido = 17603633985
if game.PlaceId ~= PlaceIdPermitido then
    -- Expulsar o usuário se não estiver no jogo suportado
    -- Exibe a mensagem e chama o TeleportService:Kick
    local TeleportService = game:GetService("TeleportService")
    TeleportService:Kick("Este Hub não suporta essa experiência.")
    return
end

-----------------------------------
-- Serviços e Configurações Básicas
-----------------------------------
local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local TweenService = game:GetService("TweenService")
local player = Players.LocalPlayer
local character = player.Character or player.CharacterAdded:Wait()

-- Valores padrões para o Aimbot (você pode ajustar)
local DEFAULT_FOV = 70      -- FOV do aimbot (vamos diminuir para maior precisão)
local DEFAULT_SMOOTH = 0.2  -- Valor de suavização (quanto menor, mais rápida a mira)

-----------------------------------
-- Criação do ScreenGui
-----------------------------------
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "AimbotHub"
ScreenGui.Parent = player:WaitForChild("PlayerGui")

-----------------------------------
-- MainFrame (Interface principal)
-----------------------------------
local MainFrame = Instance.new("Frame")
MainFrame.Name = "MainFrame"
MainFrame.Size = UDim2.new(0, 400, 0, 300)
MainFrame.Position = UDim2.new(0.5, -200, 0.5, -150)
MainFrame.BackgroundColor3 = Color3.fromRGB(30, 30, 40)
MainFrame.BorderSizePixel = 0
MainFrame.Parent = ScreenGui

local MainFrameCorner = Instance.new("UICorner")
MainFrameCorner.CornerRadius = UDim.new(0, 12)
MainFrameCorner.Parent = MainFrame

-----------------------------------
-- Cabeçalho
-----------------------------------
local Header = Instance.new("Frame")
Header.Name = "Header"
Header.Size = UDim2.new(1, 0, 0, 40)
Header.BackgroundColor3 = Color3.fromRGB(25, 25, 35)
Header.BorderSizePixel = 0
Header.Parent = MainFrame

local HeaderCorner = Instance.new("UICorner")
HeaderCorner.CornerRadius = UDim.new(0, 12)
HeaderCorner.Parent = Header

local Title = Instance.new("TextLabel")
Title.Name = "Title"
Title.Size = UDim2.new(1, -60, 1, 0)
Title.Position = UDim2.new(0, 10, 0, 0)
Title.BackgroundTransparency = 1
Title.Text = "Aimbot Hub v2.4"
Title.TextColor3 = Color3.fromRGB(255,255,255)
Title.Font = Enum.Font.GothamBold
Title.TextSize = 18
Title.Parent = Header

-----------------------------------
-- Botões de Aba (Aimbot e ESP)
-----------------------------------
-- Botão Aimbot
local AimbotTab = Instance.new("TextButton")
AimbotTab.Name = "AimbotTab"
AimbotTab.Size = UDim2.new(0, 50, 0, 40)
AimbotTab.Position = UDim2.new(1, -100, 0, 0)
AimbotTab.BackgroundColor3 = Color3.fromRGB(80, 0, 120)
AimbotTab.Text = "AIM"
AimbotTab.TextColor3 = Color3.new(1,1,1)
AimbotTab.Font = Enum.Font.GothamBold
AimbotTab.TextSize = 14
AimbotTab.Parent = Header

-- Botão ESP
local ESPTab = Instance.new("TextButton")
ESPTab.Name = "ESPTab"
ESPTab.Size = UDim2.new(0, 50, 0, 40)
ESPTab.Position = UDim2.new(1, -50, 0, 0)
ESPTab.BackgroundColor3 = Color3.fromRGB(80, 0, 120)
ESPTab.Text = "ESP"
ESPTab.TextColor3 = Color3.new(1,1,1)
ESPTab.Font = Enum.Font.GothamBold
ESPTab.TextSize = 14
ESPTab.Parent = Header

-----------------------------------
-- Painel de Conteúdo (onde a aba ativa aparece)
-----------------------------------
local ContentFrame = Instance.new("Frame")
ContentFrame.Name = "ContentFrame"
ContentFrame.Size = UDim2.new(1, 0, 1, -40)
ContentFrame.Position = UDim2.new(0, 0, 0, 40)
ContentFrame.BackgroundColor3 = Color3.fromRGB(30, 30, 40)
ContentFrame.Parent = MainFrame

-----------------------------------
-- Criação das abas individuais
-----------------------------------

-------------------------
-- Aba Aimbot
-------------------------
local AimbotFrame = Instance.new("Frame")
AimbotFrame.Name = "AimbotFrame"
AimbotFrame.Size = UDim2.new(1, 0, 1, 0)
AimbotFrame.BackgroundTransparency = 1
AimbotFrame.Parent = ContentFrame

-- Label de status Aimbot
local AimbotStatus = Instance.new("TextLabel")
AimbotStatus.Name = "AimbotStatus"
AimbotStatus.Size = UDim2.new(1, -20, 0, 30)
AimbotStatus.Position = UDim2.new(0, 10, 0, 10)
AimbotStatus.BackgroundTransparency = 1
AimbotStatus.Text = "Aimbot: OFF"
AimbotStatus.TextColor3 = Color3.new(1,1,1)
AimbotStatus.Font = Enum.Font.GothamBold
AimbotStatus.TextSize = 16
AimbotStatus.Parent = AimbotFrame

-- Botão Toggle Aimbot
local ToggleAimbot = Instance.new("TextButton")
ToggleAimbot.Name = "ToggleAimbot"
ToggleAimbot.Size = UDim2.new(0, 100, 0, 30)
ToggleAimbot.Position = UDim2.new(0, 10, 0, 50)
ToggleAimbot.BackgroundColor3 = Color3.fromRGB(80, 0, 120)
ToggleAimbot.Text = "Ativar"
ToggleAimbot.TextColor3 = Color3.new(1,1,1)
ToggleAimbot.Font = Enum.Font.GothamBold
ToggleAimbot.TextSize = 14
ToggleAimbot.Parent = AimbotFrame

local aimbotEnabled = false
ToggleAimbot.MouseButton1Click:Connect(function()
    aimbotEnabled = not aimbotEnabled
    if aimbotEnabled then
        ToggleAimbot.Text = "Desativar"
        AimbotStatus.Text = "Aimbot: ON"
        -- Aqui você coloca sua lógica de aimbot (lock-on, smooth tracking, etc.)
    else
        ToggleAimbot.Text = "Ativar"
        AimbotStatus.Text = "Aimbot: OFF"
        -- Desativar a mira automática
    end
end)

-- Configuração de FOV (apenas o valor)
local FOVLabel = Instance.new("TextLabel")
FOVLabel.Name = "FOVLabel"
FOVLabel.Size = UDim2.new(0, 120, 0, 20)
FOVLabel.Position = UDim2.new(0, 10, 0, 90)
FOVLabel.BackgroundTransparency = 1
FOVLabel.Text = "FOV: " .. tostring(DEFAULT_FOV)
FOVLabel.TextColor3 = Color3.new(1,1,1)
FOVLabel.Font = Enum.Font.Gotham
FOVLabel.TextSize = 14
FOVLabel.Parent = AimbotFrame

-- Input para FOV
local FOVBox = Instance.new("TextBox")
FOVBox.Name = "FOVBox"
FOVBox.Size = UDim2.new(0, 50, 0, 20)
FOVBox.Position = UDim2.new(0, 140, 0, 90)
FOVBox.BackgroundColor3 = Color3.fromRGB(50, 0, 70)
FOVBox.Text = tostring(DEFAULT_FOV)
FOVBox.TextColor3 = Color3.new(1,1,1)
FOVBox.Font = Enum.Font.GothamBold
FOVBox.TextSize = 14
FOVBox.Parent = AimbotFrame

local ApplyFOV = Instance.new("TextButton")
ApplyFOV.Name = "ApplyFOV"
ApplyFOV.Size = UDim2.new(0, 60, 0, 20)
ApplyFOV.Position = UDim2.new(0, 200, 0, 90)
ApplyFOV.BackgroundColor3 = Color3.fromRGB(80, 0, 120)
ApplyFOV.Text = "Aplicar"
ApplyFOV.TextColor3 = Color3.new(1,1,1)
ApplyFOV.Font = Enum.Font.GothamBold
ApplyFOV.TextSize = 14
ApplyFOV.Parent = AimbotFrame

ApplyFOV.MouseButton1Click:Connect(function()
    local newFOV = tonumber(FOVBox.Text)
    if newFOV then
        DEFAULT_FOV = newFOV
        FOVLabel.Text = "FOV: " .. tostring(DEFAULT_FOV)
        -- Atualize a lógica do aimbot para usar o novo FOV, se necessário.
    end
end)

-- Configuração de Suavização
local SmoothLabel = Instance.new("TextLabel")
SmoothLabel.Name = "SmoothLabel"
SmoothLabel.Size = UDim2.new(0, 120, 0, 20)
SmoothLabel.Position = UDim2.new(0, 10, 0, 120)
SmoothLabel.BackgroundTransparency = 1
SmoothLabel.Text = "Smooth: " .. tostring(DEFAULT_SMOOTH)
SmoothLabel.TextColor3 = Color3.new(1,1,1)
SmoothLabel.Font = Enum.Font.Gotham
SmoothLabel.TextSize = 14
SmoothLabel.Parent = AimbotFrame

local SmoothBox = Instance.new("TextBox")
SmoothBox.Name = "SmoothBox"
SmoothBox.Size = UDim2.new(0, 50, 0, 20)
SmoothBox.Position = UDim2.new(0, 140, 0, 120)
SmoothBox.BackgroundColor3 = Color3.fromRGB(50, 0, 70)
SmoothBox.Text = tostring(DEFAULT_SMOOTH)
SmoothBox.TextColor3 = Color3.new(1,1,1)
SmoothBox.Font = Enum.Font.GothamBold
SmoothBox.TextSize = 14
SmoothBox.Parent = AimbotFrame

local ApplySmooth = Instance.new("TextButton")
ApplySmooth.Name = "ApplySmooth"
ApplySmooth.Size = UDim2.new(0, 60, 0, 20)
ApplySmooth.Position = UDim2.new(0, 200, 0, 120)
ApplySmooth.BackgroundColor3 = Color3.fromRGB(80, 0, 120)
ApplySmooth.Text = "Aplicar"
ApplySmooth.TextColor3 = Color3.new(1,1,1)
ApplySmooth.Font = Enum.Font.GothamBold
ApplySmooth.TextSize = 14
ApplySmooth.Parent = AimbotFrame

ApplySmooth.MouseButton1Click:Connect(function()
    local newSmooth = tonumber(SmoothBox.Text)
    if newSmooth then
        DEFAULT_SMOOTH = newSmooth
        SmoothLabel.Text = "Smooth: " .. tostring(DEFAULT_SMOOTH)
        -- Atualize a lógica do aimbot para usar o novo fator de suavização.
    end
end)

-------------------------
-- Aba ESP
-------------------------
local ESPFrame = Instance.new("Frame")
ESPFrame.Name = "ESPFrame"
ESPFrame.Size = UDim2.new(1, 0, 1, 0)
ESPFrame.BackgroundTransparency = 1
ESPFrame.Visible = false  -- Por padrão, inicie com a aba Aimbot ativa
ESPFrame.Parent = ContentFrame

-- Label de status ESP
local ESPStatus = Instance.new("TextLabel")
ESPStatus.Name = "ESPStatus"
ESPStatus.Size = UDim2.new(1, -20, 0, 30)
ESPStatus.Position = UDim2.new(0, 10, 0, 10)
ESPStatus.BackgroundTransparency = 1
ESPStatus.Text = "ESP: OFF"
ESPStatus.TextColor3 = Color3.new(1,1,1)
ESPStatus.Font = Enum.Font.GothamBold
ESPStatus.TextSize = 16
ESPStatus.Parent = ESPFrame

-- Botão Toggle ESP
local ToggleESP = Instance.new("TextButton")
ToggleESP.Name = "ToggleESP"
ToggleESP.Size = UDim2.new(0, 100, 0, 30)
ToggleESP.Position = UDim2.new(0, 10, 0, 50)
ToggleESP.BackgroundColor3 = Color3.fromRGB(80, 0, 120)
ToggleESP.Text = "Ativar"
ToggleESP.TextColor3 = Color3.new(1,1,1)
ToggleESP.Font = Enum.Font.GothamBold
ToggleESP.TextSize = 14
ToggleESP.Parent = ESPFrame

local espEnabled = false
ToggleESP.MouseButton1Click:Connect(function()
    espEnabled = not espEnabled
    if espEnabled then
        ToggleESP.Text = "Desativar"
        ESPStatus.Text = "ESP: ON"
        -- Ativar lógica do ESP: escolha o modo conforme a opção selecionada abaixo.
    else
        ToggleESP.Text = "Ativar"
        ESPStatus.Text = "ESP: OFF"
        -- Desativar lógica do ESP.
        -- Aqui você deve limpar quaisquer indicadores ou marcas que o ESP estava aplicando.
    end
end)

-- Opção de Modos ESP (com nome ou sem nome)
local ESPModeLabel = Instance.new("TextLabel")
ESPModeLabel.Name = "ESPModeLabel"
ESPModeLabel.Size = UDim2.new(0, 180, 0, 20)
ESPModeLabel.Position = UDim2.new(0, 10, 0, 90)
ESPModeLabel.BackgroundTransparency = 1
ESPModeLabel.Text = "Modo ESP: (1) Com Nome, (2) Sem Nome"
ESPModeLabel.TextColor3 = Color3.new(1,1,1)
ESPModeLabel.Font = Enum.Font.Gotham
ESPModeLabel.TextSize = 14
ESPModeLabel.Parent = ESPFrame

local espMode = 1  -- 1 para ESP com nome, 2 para ESP sem nome

local ESPModeButton = Instance.new("TextButton")
ESPModeButton.Name = "ESPModeButton"
ESPModeButton.Size = UDim2.new(0, 100, 0, 25)
ESPModeButton.Position = UDim2.new(0, 10, 0, 120)
ESPModeButton.BackgroundColor3 = Color3.fromRGB(80,0,120)
ESPModeButton.Text = "Alternar Modo"
ESPModeButton.TextColor3 = Color3.new(1,1,1)
ESPModeButton.Font = Enum.Font.GothamBold
ESPModeButton.TextSize = 14
ESPModeButton.Parent = ESPFrame

ESPModeButton.MouseButton1Click:Connect(function()
    if espMode == 1 then
        espMode = 2
        ESPModeLabel.Text = "Modo ESP: Sem Nome"
    else
        espMode = 1
        ESPModeLabel.Text = "Modo ESP: Com Nome"
    end
    -- Atualize a lógica do ESP para usar o modo selecionado.
end)

-- (Exemplo simples) Loop para aplicar ESP – aqui você deve integrar seu código de marcação
spawn(function()
    while wait(0.5) do
        if espEnabled then
            for _, plr in pairs(Players:GetPlayers()) do
                if plr ~= player and plr.Character and plr.Character:FindFirstChild("Head") then
                    local head = plr.Character.Head
                    if espMode == 1 then
                        -- ESP COM NOME: Marque a cabeça e exiba o nome (exemplo simples)
                        head.BrickColor = BrickColor.new("Bright red")
                        -- Você pode criar um BillboardGui para exibir o nome acima da cabeça
                        if not head:FindFirstChild("ESPName") then
                            local billboard = Instance.new("BillboardGui", head)
                            billboard.Name = "ESPName"
                            billboard.Size = UDim2.new(0, 100, 0, 30)
                            billboard.Adornee = head
                            billboard.AlwaysOnTop = true
                            local namelabel = Instance.new("TextLabel", billboard)
                            namelabel.Size = UDim2.new(1, 0, 1, 0)
                            namelabel.BackgroundTransparency = 1
                            namelabel.Text = plr.Name
                            namelabel.TextColor3 = Color3.new(1,1,1)
                            namelabel.Font = Enum.Font.GothamBold
                            namelabel.TextSize = 14
                        end
                    else
                        -- ESP SEM NOME: apenas mude a cor da cabeça; se existir BillboardGui, remova-o.
                        head.BrickColor = BrickColor.new("Bright red")
                        if head:FindFirstChild("ESPName") then
                            head.ESPName:Destroy()
                        end
                    end
                end
            end
        else
            -- Quando desligado, opcionalmente você pode restaurar as cores originais e remover Billboards
            for _, plr in pairs(Players:GetPlayers()) do
                if plr ~= player and plr.Character and plr.Character:FindFirstChild("Head") then
                    local head = plr.Character.Head
                    head.BrickColor = BrickColor.new("Medium stone grey")
                    if head:FindFirstChild("ESPName") then
                        head.ESPName:Destroy()
                    end
                end
            end
        end
    end
end)

-----------------------------------
-- Alternando abas através dos Botões do Header
-----------------------------------
AimbotTab.MouseButton1Click:Connect(function()
    AimbotFrame.Visible = true
    ESPFrame.Visible = false
    AimbotTab.BackgroundColor3 = Color3.fromRGB(100, 0, 140)
    ESPTab.BackgroundColor3 = Color3.fromRGB(80, 0, 120)
end)

ESPTab.MouseButton1Click:Connect(function()
    AimbotFrame.Visible = false
    ESPFrame.Visible = true
    ESPTab.BackgroundColor3 = Color3.fromRGB(100, 0, 140)
    AimbotTab.BackgroundColor3 = Color3.fromRGB(80, 0, 120)
end)

-----------------------------------
-- Movimentação do MainFrame (arrastável pelo Header)
-----------------------------------
local dragging = false
local dragOffset = Vector2.new()

Header.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        dragging = true
        dragOffset = MainFrame.AbsolutePosition - input.Position
        input.Changed:Connect(function()
            if input.UserInputState == Enum.UserInputState.End then
                dragging = false
            end
        end)
    end
end)

UserInputService.InputChanged:Connect(function(input)
    if dragging and input.UserInputType == Enum.UserInputType.MouseMovement then
        MainFrame.Position = UDim2.new(0, input.Position.X + dragOffset.X, 0, input.Position.Y + dragOffset.Y)
    end
end)

-----------------------------------
-- Redimensionamento do MainFrame (com limite mínimo)
-----------------------------------
local resizing = false
local resizeStart = Vector2.new()
local initialSize = MainFrame.AbsoluteSize

local resizeHandle = Instance.new("Frame")
resizeHandle.Name = "ResizeHandle"
resizeHandle.Size = UDim2.new(0, 20, 0, 20)
resizeHandle.AnchorPoint = Vector2.new(1, 1)
resizeHandle.Position = UDim2.new(1, 0, 1, 0)
resizeHandle.BackgroundTransparency = 1
resizeHandle.Parent = MainFrame

local resizeIcon = Instance.new("TextLabel")
resizeIcon.Name = "ResizeIcon"
resizeIcon.Size = UDim2.new(1, 0, 1, 0)
resizeIcon.BackgroundTransparency = 1
resizeIcon.Text = "↘"
resizeIcon.TextColor3 = Color3.new(1,1,1)
resizeIcon.Font = Enum.Font.GothamBold
resizeIcon.TextScaled = true
resizeIcon.Parent = resizeHandle

resizeHandle.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        resizing = true
        resizeStart = input.Position
        initialSize = MainFrame.AbsoluteSize
        input.Changed:Connect(function()
            if input.UserInputState == Enum.UserInputState.End then
                resizing = false
            end
        end)
    end
end)

UserInputService.InputChanged:Connect(function(input)
    if resizing and input.UserInputType == Enum.UserInputType.MouseMovement then
        local delta = input.Position - resizeStart
        local newWidth = math.max(initialSize.X + delta.X, 400)
        local newHeight = math.max(initialSize.Y + delta.Y, 300)
        MainFrame.Size = UDim2.new(0, newWidth, 0, newHeight)
    end
end)

-----------------------------------
-- Fim da Interface
-----------------------------------
print("Aimbot Hub v2.4 carregado para o jogo com ID " .. tostring(PlaceIdPermitido))

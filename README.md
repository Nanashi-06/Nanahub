--// CONFIGURAÇÕES DO HUB
local HUB_NAME = "Nana Hub Brookhaven Edition"
local HUB_VERSION = "v1.0"

--// SERVIÇOS
local UserInputService = game:GetService("UserInputService")

--// CRIAÇÃO DO SCREENGUI PRINCIPAL
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "NanaHubUI"
ScreenGui.ResetOnSpawn = false
ScreenGui.Parent = game.Players.LocalPlayer:WaitForChild("PlayerGui")

--// MAINFRAME (Container da Interface) - Tamanho Inicial Compacto
local mainFrame = Instance.new("Frame")
mainFrame.Name = "MainFrame"
mainFrame.Size = UDim2.new(0, 550, 0, 400)
mainFrame.Position = UDim2.new(0.5, -275, 0.5, -200)
mainFrame.BackgroundColor3 = Color3.fromRGB(30, 0, 50)   -- tom roxo escuro
mainFrame.BorderSizePixel = 0
mainFrame.Parent = ScreenGui

local mainUICorner = Instance.new("UICorner")
mainUICorner.CornerRadius = UDim.new(0, 10)
mainUICorner.Parent = mainFrame

-------------------------------------------------------------------
-- CABEÇALHO: Exibe o título e a versão (dentro do Hub)
-------------------------------------------------------------------
local header = Instance.new("Frame")
header.Name = "Header"
header.Size = UDim2.new(1, 0, 0, 40)
header.BackgroundColor3 = Color3.fromRGB(60, 0, 100)  -- roxo mais vivo
header.BorderSizePixel = 0
header.Parent = mainFrame

local headerUICorner = Instance.new("UICorner")
headerUICorner.CornerRadius = UDim.new(0, 10)
headerUICorner.Parent = header

-- Título (nome + versão)
local titleLabel = Instance.new("TextLabel")
titleLabel.Name = "TitleLabel"
titleLabel.Size = UDim2.new(1, -50, 1, 0)
titleLabel.Position = UDim2.new(0, 10, 0, 0)
titleLabel.BackgroundTransparency = 1
titleLabel.Text = HUB_NAME .. " " .. HUB_VERSION
titleLabel.TextColor3 = Color3.new(1, 1, 1)
titleLabel.Font = Enum.Font.GothamBold
titleLabel.TextSize = 18
titleLabel.Parent = header

-------------------------------------------------------------------
-- MENU LATERAL PARA CATEGORIAS
-------------------------------------------------------------------
local menuFrame = Instance.new("Frame")
menuFrame.Name = "MenuFrame"
menuFrame.Size = UDim2.new(0, 150, 1, -40)  -- desconta a altura do cabeçalho
menuFrame.Position = UDim2.new(0, 0, 0, 40)
menuFrame.BackgroundColor3 = Color3.fromRGB(20, 0, 40)
menuFrame.BorderSizePixel = 0
menuFrame.Parent = mainFrame

local menuUICorner = Instance.new("UICorner")
menuUICorner.CornerRadius = UDim.new(0, 10)
menuUICorner.Parent = menuFrame

local categories = {"Tp & View", "Troll", "Carro", "Avatar", "Utilidades"}
local menuButtons = {}

for i, cat in ipairs(categories) do
    local btn = Instance.new("TextButton")
    btn.Name = cat.."Button"
    btn.Size = UDim2.new(1, -20, 0, 35)
    btn.Position = UDim2.new(0, 10, 0, 10 + (i-1) * 40)
    btn.Text = cat
    btn.TextColor3 = Color3.fromRGB(255, 255, 255)
    btn.BackgroundColor3 = Color3.fromRGB(50, 0, 70)
    btn.Font = Enum.Font.GothamBold
    btn.TextSize = 16
    btn.BorderSizePixel = 0
    btn.Parent = menuFrame

    local btnCorner = Instance.new("UICorner")
    btnCorner.CornerRadius = UDim.new(0, 5)
    btnCorner.Parent = btn

    table.insert(menuButtons, btn)
end

-------------------------------------------------------------------
-- PAINEL DE CONTEÚDO PARA AS FUNÇÕES
-------------------------------------------------------------------
local contentFrame = Instance.new("Frame")
contentFrame.Name = "ContentFrame"
contentFrame.Size = UDim2.new(1, -150, 1, -40)
contentFrame.Position = UDim2.new(0, 150, 0, 40)
contentFrame.BackgroundColor3 = Color3.fromRGB(30, 0, 60)
contentFrame.BorderSizePixel = 0
contentFrame.Parent = mainFrame

local contentUICorner = Instance.new("UICorner")
contentUICorner.CornerRadius = UDim.new(0, 10)
contentUICorner.Parent = contentFrame

local listLayout = Instance.new("UIListLayout")
listLayout.Padding = UDim.new(0, 8)
listLayout.SortOrder = Enum.SortOrder.LayoutOrder
listLayout.Parent = contentFrame

-------------------------------------------------------------------
-- DEFINIÇÃO DAS FUNÇÕES POR CATEGORIA
-------------------------------------------------------------------
local categoryFunctions = {
    ["Tp & View"] = {"Teleport", "Free Camera", "Third Person"},
    Troll = {"Fling", "Random Explode", "Swap Positions"},
    Carro = {"Speed Boost", "Vehicle Teleport", "Drift Mode"},
    Avatar = {"Change Outfit", "Resize Character", "Colorize Avatar"},
    Utilidades = {"ESP", "Admin Detector", "Anti-Kick"}
}

local function populateContent(category)
    -- Remove botões anteriores (exceto o UIListLayout)
    for _, child in ipairs(contentFrame:GetChildren()) do
        if child:IsA("TextButton") then
            child:Destroy()
        end
    end

    for _, funcName in ipairs(categoryFunctions[category] or {}) do
        local funcButton = Instance.new("TextButton")
        funcButton.Name = funcName.."Button"
        funcButton.Size = UDim2.new(1, -20, 0, 30)
        funcButton.BackgroundColor3 = Color3.fromRGB(70, 0, 90)
        funcButton.TextColor3 = Color3.fromRGB(255, 255, 255)
        funcButton.Font = Enum.Font.Gotham
        funcButton.TextSize = 14
        funcButton.Text = funcName
        funcButton.Parent = contentFrame

        local btnCorner = Instance.new("UICorner")
        btnCorner.CornerRadius = UDim.new(0, 5)
        btnCorner.Parent = funcButton

        funcButton.MouseButton1Click:Connect(function()
            print("Função executada: " .. funcName)
            -- Coloque aqui a lógica da função que desejar.
        end)
    end
end

-------------------------------------------------------------------
-- CONEXÃO DOS BOTÕES DO MENU PARA ATUALIZAR O CONTEÚDO
-------------------------------------------------------------------
for _, btn in ipairs(menuButtons) do
    btn.MouseButton1Click:Connect(function()
        populateContent(btn.Text)
    end)
end

-- Inicialmente, exibe as funções da primeira categoria ("Tp & View")
populateContent("Tp & View")

-------------------------------------------------------------------
-- BOTÃO DE MINIMIZAR/RESTAURAR - FICA FORA DO MAINFRAME
-------------------------------------------------------------------
local outsideMinimize = Instance.new("TextButton")
outsideMinimize.Name = "OutsideMinimize"
outsideMinimize.Size = UDim2.new(0, 30, 0, 30)
-- Posiciona o botão no canto superior direito da tela (pode ajustar conforme necessário)
outsideMinimize.Position = UDim2.new(1, 60, 0, 10)
outsideMinimize.BackgroundColor3 = Color3.fromRGB(100, 0, 150)
outsideMinimize.Text = "-"
outsideMinimize.TextColor3 = Color3.new(1, 1, 1)
outsideMinimize.Font = Enum.Font.GothamBold
outsideMinimize.TextSize = 24
outsideMinimize.BorderSizePixel = 0
outsideMinimize.Parent = ScreenGui

local isHubVisible = true

outsideMinimize.MouseButton1Click:Connect(function()
    if isHubVisible then
        mainFrame.Visible = false
        outsideMinimize.Text = "+"
    else
        mainFrame.Visible = true
        outsideMinimize.Text = "-"
    end
    isHubVisible = not isHubVisible
end)

-------------------------------------------------------------------
-- REDIMENSIONAMENTO DO HUB (ARRASTE O CANTO INFERIOR DIREITO)
-------------------------------------------------------------------
local resizeHandle = Instance.new("Frame")
resizeHandle.Name = "ResizeHandle"
resizeHandle.Size = UDim2.new(0, 20, 0, 20)
resizeHandle.AnchorPoint = Vector2.new(1, 1)
resizeHandle.Position = UDim2.new(1, 0, 1, 0)
resizeHandle.BackgroundTransparency = 1
resizeHandle.Parent = mainFrame

local resizeIcon = Instance.new("TextLabel")
resizeIcon.Name = "ResizeIcon"
resizeIcon.Size = UDim2.new(1, 0, 1, 0)
resizeIcon.BackgroundTransparency = 1
resizeIcon.Text = "↘"
resizeIcon.TextColor3 = Color3.fromRGB(255, 255, 255)
resizeIcon.Font = Enum.Font.GothamBold
resizeIcon.TextScaled = true
resizeIcon.Parent = resizeHandle

local dragging = false
local dragStart, startSize

resizeHandle.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
        dragging = true
        dragStart = input.Position
        startSize = mainFrame.AbsoluteSize
        input.Changed:Connect(function()
            if input.UserInputState == Enum.UserInputState.End then
                dragging = false
            end
        end)
    end
end)

UserInputService.InputChanged:Connect(function(input)
    if dragging and (input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch) then
        local delta = input.Position - dragStart
        local newSize = startSize + Vector2.new(delta.X, delta.Y)
        newSize = Vector2.new(math.max(newSize.X, 300), math.max(newSize.Y, 200)) -- Tamanho mínimo
        mainFrame.Size = UDim2.new(0, newSize.X, 0, newSize.Y)
    end
end)

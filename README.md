---------------------------------------------------------------------
-- NANA HUB v1.1 (BETA) com Highfield UI
-- Baseada na source v2.2 com melhorias em ESP e Aimbot,
-- agora utilizando a Highfield UI.
-- Coloque este script em StarterPlayerScripts.
---------------------------------------------------------------------

--[[
  INTEGRAÇÃO DA HIGHFIELD UI:
  
  Certifique-se de que o código abaixo (Highfield UI) esteja carregado.
  Se você já o tem em um módulo separado, basta fazer:
  
      local library = require(path_para_o_modulo)
  
  Aqui, suponhamos que o código da Highfield UI (o que você enviou) já está presente.
  
  O retorno da função Library é um objeto que expõe:
    • library:CreateWindow(WName) – que retorna um "ui" com funções:
         ui:Button(name,callback)
         ui:Slider(name, min, max, precise, callback)
         ui:Toggle(name, callback)
         ui:Box(name, callback)
         ui:Resize()  (usado internamente)
  
  Vamos usar essa API para criar a nossa janela de configurações.
]]--

-- Carrega a Highfield UI
-- Se a Highfield UI não estiver incluída em um módulo, você pode colar o código dela antes deste script
local HighfieldUILibrary = loadstring(game:HttpGet("https://raw.githubusercontent.com/YourRepo/HighfieldUI/main/source"))() 
-- Caso já tenha o código (através do arquivo "Highfield UI.txt"), você pode fazer:
-- local library = require(path_para_highfieldui) 
local library = HighfieldUILibrary  -- renomeando para "library" para compatibilidade

---------------------------------------------------------------------
-- SERVIÇOS E VARIÁVEIS INICIAIS
---------------------------------------------------------------------
local Players    = game:GetService("Players")
local RunService = game:GetService("RunService")
local UIS        = game:GetService("UserInputService")
local Camera     = workspace.CurrentCamera
local LP         = Players.LocalPlayer

local targetParts = { "Head", "HumanoidRootPart", "UpperTorso", "Torso" }

-- CONFIGURAÇÕES DO HUB
local cfg = {
	esp             = false,
	aimbot          = false,
	teamCheck       = true,
	targetIndex     = 1,       -- Inicia na "Head"
	fov             = 25,
	smoothness      = 0.5,     -- Valor mais alto para resposta mais rápida
	showFOV         = false,
	lowPower        = false,   -- Se true, atualiza apenas a cada 0.1 s
	espFillColor    = Color3.fromRGB(175, 0, 255),
	_dtAccumulator  = 0,
}
_G.NanaHub = cfg  -- Variável global para depuração, se necessário

---------------------------------------------------------------------
-- FUNÇÕES DE UTILIDADE
---------------------------------------------------------------------
local function isEnemy(plr)
	if plr == LP or not plr.Character then return false end
	if cfg.teamCheck and plr.Team == LP.Team then return false end
	return plr.Character:FindFirstChild(targetParts[cfg.targetIndex]) ~= nil
end

local function angleBetween(v1, v2)
	return math.deg(math.acos(math.clamp(v1:Dot(v2), -1, 1)))
end

---------------------------------------------------------------------
-- ESP (UTILIZANDO Highlight)
---------------------------------------------------------------------
local function addHL(char)
	if not char or char:FindFirstChild("NanaHL") then return end
	local hl = Instance.new("Highlight")
	hl.Name = "NanaHL"
	hl.FillColor = cfg.espFillColor
	hl.FillTransparency = 0.5
	hl.OutlineTransparency = 0
	hl.DepthMode = Enum.HighlightDepthMode.AlwaysOnTop
	hl.Adornee = char
	hl.Parent = char
end

local function rmHL(char)
	local h = char and char:FindFirstChild("NanaHL")
	if h then h:Destroy() end
end

local function refreshESP()
	for _, plr in ipairs(Players:GetPlayers()) do
		if plr.Character then
			if cfg.esp and isEnemy(plr) then
				if not plr.Character:FindFirstChild("NanaHL") then
					addHL(plr.Character)
				end
			else
				if plr.Character:FindFirstChild("NanaHL") then
					rmHL(plr.Character)
				end
			end
		end
	end
end

Players.PlayerAdded:Connect(function(p)
	p.CharacterAdded:Connect(function(c)
		if cfg.esp then
			task.wait(0.1)
			if isEnemy(p) then
				addHL(c)
			end
		end
	end)
end)

---------------------------------------------------------------------
-- AIMBOT – BUSCA DO ALVO MAIS PRÓXIMO
---------------------------------------------------------------------
local function getClosest()
	local best, bestAngle
	local camPos  = Camera.CFrame.Position
	local lookVec = Camera.CFrame.LookVector
	for _, plr in ipairs(Players:GetPlayers()) do
		if isEnemy(plr) then
			local part = plr.Character:FindFirstChild(targetParts[cfg.targetIndex])
			if part and part:IsA("BasePart") then
				local dir = (part.Position - camPos).Unit
				local a = angleBetween(lookVec, dir)
				if a <= cfg.fov and (not bestAngle or a < bestAngle) then
					bestAngle = a
					best = part
				end
			end
		end
	end
	return best
end

---------------------------------------------------------------------
-- VISUALIZAÇÃO DO FOV (usando a API Drawing, se disponível)
---------------------------------------------------------------------
local fovCircle
pcall(function()
	fovCircle = Drawing.new("Circle")
	fovCircle.Radius = 0
	fovCircle.Filled = false
	fovCircle.Color = Color3.fromRGB(175, 0, 255)
	fovCircle.Thickness = 1.5
end)

local function updateFOV()
	if not fovCircle then return end
	fovCircle.Visible = cfg.showFOV
	if cfg.showFOV then
		fovCircle.Radius = math.tan(math.rad(cfg.fov)) * Camera.ViewportSize.Y
		fovCircle.Position = Vector2.new(Camera.ViewportSize.X / 2, Camera.ViewportSize.Y / 2)
	end
end

---------------------------------------------------------------------
-- CRIAÇÃO DA INTERFACE COM HIGHFIELD UI
---------------------------------------------------------------------
local ui = library:CreateWindow("NANA HUB v1.1 (BETA)")  
-- Essa função cria a janela principal; o retorno (ui) contém
-- métodos para adicionar controles (Button, Slider, Toggle, Box).

-- Adiciona um botão de teste para validar a interface
ui:Button("Teste Botão", function()
	print("Botão Teste Funcionou!")
end)

-- Cria os controles para as configurações:
ui:Toggle("ESP", function(Value)
	cfg.esp = Value
end)

ui:Toggle("Aimbot", function(Value)
	cfg.aimbot = Value
end)

ui:Toggle("Verificar Time", function(Value)
	cfg.teamCheck = Value
end)

ui:Slider("FOV", 0, 180, false, function(Value)
	cfg.fov = Value
end)

ui:Slider("Smoothness", 0, 1, false, function(Value)
	cfg.smoothness = Value
end)

ui:Toggle("Mostrar FOV", function(Value)
	cfg.showFOV = Value
end)

ui:Toggle("Modo Leve (Low Power)", function(Value)
	cfg.lowPower = Value
end)

ui:Button("Ciclar Parte-Alvo", function()
	cfg.targetIndex = cfg.targetIndex % #targetParts + 1
	-- Notifique o usuário; aqui usamos print para depuração
	print("Parte-Alvo: " .. targetParts[cfg.targetIndex])
end)

ui:Button("Fechar Hub", function()
	-- Para fechar a interface, podemos simplesmente desabilitar o ScreenGui
	-- ou destruir o objeto criado.
	game:GetService("CoreGui"):FindFirstChild("UI Library"):Destroy()
end)

---------------------------------------------------------------------
-- LOOP PRINCIPAL – ATUALIZA ESP, AIMBOT E FOV
---------------------------------------------------------------------
RunService.RenderStepped:Connect(function(dt)
	cfg._dtAccumulator = cfg._dtAccumulator + dt
	updateFOV()
	if cfg.lowPower and cfg._dtAccumulator < 0.1 then return end
	cfg._dtAccumulator = 0
	refreshESP()
	if cfg.aimbot then
		local tgt = getClosest()
		if tgt then
			local camCF = Camera.CFrame
			local dir = (tgt.Position - camCF.Position).Unit
			local goal = CFrame.new(camCF.Position, camCF.Position + dir)
			local alpha = math.clamp(cfg.smoothness * dt * 60, 0, 1)
			Camera.CFrame = camCF:Lerp(goal, alpha)
		end
	end
end)

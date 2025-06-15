---------------------------------------------------------------------
-- 🟣  NANA HUB  v2.2  –  Modo Compatibilidade + Modo Leve opcional
-- Coloque como LocalScript em StarterPlayerScripts.
---------------------------------------------------------------------
local Players      = game:GetService("Players")
local RunService   = game:GetService("RunService")
local UIS          = game:GetService("UserInputService")
local Camera       = workspace.CurrentCamera
local LP           = Players.LocalPlayer

---------------------------------------------------------------------
-- 🔧 CONFIGURAÇÃO (pode editar depois)
---------------------------------------------------------------------
local targetParts = { "Head", "HumanoidRootPart", "UpperTorso", "Torso" }

local cfg = {
    esp             = false,
    aimbot          = false,
    teamCheck       = true,
    targetIndex     = 1,                -- começa em "Head"
    fov             = 25,
    smoothness      = 0.15,
    showFOV         = false,
    lowPower        = false,            -- se true, só atualiza a cada 0.1 s
    espFillColor    = Color3.fromRGB(175,0,255),

    -- tempo interno
    _dtAccumulator  = 0,
}
_G.NanaHub = cfg  -- global opcional pra depuração

---------------------------------------------------------------------
-- 🖼️  INTERFACE
---------------------------------------------------------------------
local gui  = Instance.new("ScreenGui", LP:WaitForChild("PlayerGui"))
gui.Name   = "NanaHubGUI"
gui.ResetOnSpawn = false

local main = Instance.new("Frame", gui)
main.Size  = UDim2.new(0, 280, 0, 285)
main.AnchorPoint = Vector2.new(0.5,0.5)
main.Position    = UDim2.new(0.5,0,0.5,0)
main.BackgroundColor3 = Color3.fromRGB(50,0,80)
main.BackgroundTransparency = 0.05
main.BorderSizePixel = 0
Instance.new("UICorner", main).CornerRadius = UDim.new(0,10)

local title = Instance.new("TextLabel", main)
title.Size = UDim2.new(1,0,0,30)
title.Text = "NANA HUB  v2.2"
title.Font = Enum.Font.GothamBlack
title.TextSize = 18
title.TextColor3 = Color3.new(1,1,1)
title.BackgroundTransparency = 1

-- botão de minimizar
local mini = Instance.new("TextButton", gui)
mini.Size = UDim2.new(0,110,0,30)
mini.Position = UDim2.new(1,-120,1,-40)
mini.Text = "Fechar Hub"
mini.Font = Enum.Font.GothamBold
mini.TextSize = 14
mini.BackgroundColor3 = Color3.fromRGB(100,0,150)
mini.TextColor3 = Color3.new(1,1,1)
local guiVisible = true
mini.MouseButton1Click:Connect(function()
    guiVisible = not guiVisible
    main.Visible = guiVisible
    mini.Text = guiVisible and "Fechar Hub" or "Abrir Hub"
end)

-- helper p/ criar botões-toggle
local function makeToggle(label, order, key, callback)
    local btn = Instance.new("TextButton", main)
    btn.Size = UDim2.new(1,-20,0,35)
    btn.Position = UDim2.new(0,10,0,45+order*45)
    btn.Font = Enum.Font.Gotham
    btn.TextSize = 14
    btn.TextColor3 = Color3.new(1,1,1)
    btn.BackgroundColor3 = Color3.fromRGB(80,0,120)

    local function refresh()
        local on = cfg[key]
        btn.Text = ("%s: %s"):format(label, on and "Ligado" or "Desligado")
        btn.BackgroundColor3 = on and Color3.fromRGB(30,160,70) or Color3.fromRGB(80,0,120)
        if callback then callback(on) end
    end
    btn.MouseButton1Click:Connect(function()
        cfg[key] = not cfg[key]
        refresh()
    end)
    refresh()
end

-- botão para ciclar parte-alvo
local partBtn = Instance.new("TextButton", main)
partBtn.Size = UDim2.new(1,-20,0,35)
partBtn.Position = UDim2.new(0,10,0,45+3*45)
partBtn.Font = Enum.Font.Gotham
partBtn.TextSize = 14
partBtn.TextColor3 = Color3.new(1,1,1)
partBtn.BackgroundColor3 = Color3.fromRGB(120,0,180)
local function refreshPart()
    partBtn.Text = "Parte-Alvo: "..targetParts[cfg.targetIndex]
end
partBtn.MouseButton1Click:Connect(function()
    cfg.targetIndex = cfg.targetIndex % #targetParts + 1
    refreshPart()
end)
refreshPart()

makeToggle("ESP",            0, "esp")
makeToggle("Aimbot",         1, "aimbot")
makeToggle("Verificar Time", 2, "teamCheck")
makeToggle("Mostrar FOV",    4, "showFOV")
makeToggle("Modo Leve",      5, "lowPower")

---------------------------------------------------------------------
-- 🛠️  UTILIDADES
---------------------------------------------------------------------
local function isEnemy(plr)
    if plr == LP or not plr.Character then return false end
    if cfg.teamCheck and plr.Team == LP.Team then return false end
    return plr.Character:FindFirstChild(targetParts[cfg.targetIndex]) ~= nil
end
local function angleBetween(v1,v2)
    return math.deg(math.acos(math.clamp(v1:Dot(v2),-1,1)))
end

---------------------------------------------------------------------
-- 🌈  ESP (Highlight)
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
    hl.Parent  = char
end
local function rmHL(char)
    local h = char and char:FindFirstChild("NanaHL")
    if h then h:Destroy() end
end
local function refreshESP()
    for _,plr in ipairs(Players:GetPlayers()) do
        if plr.Character then
            if cfg.esp and isEnemy(plr) then addHL(plr.Character)
            else rmHL(plr.Character) end
        end
    end
end
Players.PlayerAdded:Connect(function(p)
    p.CharacterAdded:Connect(function(c)
        if cfg.esp then task.wait(0.1); if isEnemy(p) then addHL(c) end end
    end)
end)

---------------------------------------------------------------------
-- 🎯  AIMBOT
---------------------------------------------------------------------
local function getClosest()
    local best,ang
    local camPos  = Camera.CFrame.Position
    local lookVec = Camera.CFrame.LookVector
    for _,plr in ipairs(Players:GetPlayers()) do
        if isEnemy(plr) then
            local part = plr.Character:FindFirstChild(targetParts[cfg.targetIndex])
            if part and part:IsA("BasePart") then
                local dir = (part.Position - camPos).Unit
                local a   = angleBetween(lookVec, dir)
                if a <= cfg.fov and (not ang or a < ang) then
                    ang,best = a,part
                end
            end
        end
    end
    return best
end

---------------------------------------------------------------------
-- ⭕  Desenho do FOV (usando Drawing API; segura se não existir)
---------------------------------------------------------------------
local fovCircle
pcall(function()
    fovCircle = Drawing.new("Circle")
    fovCircle.Radius = 0
    fovCircle.Filled = false
    fovCircle.Color  = Color3.fromRGB(175,0,255)
    fovCircle.Thickness = 1.5
end)
local function updateFOV()
    if not fovCircle then return end
    fovCircle.Visible = cfg.showFOV
    if cfg.showFOV then
        fovCircle.Radius = math.tan(math.rad(cfg.fov))*Camera.ViewportSize.Y
        fovCircle.Position = Vector2.new(Camera.ViewportSize.X/2,Camera.ViewportSize.Y/2)
    end
end

---------------------------------------------------------------------
-- 🔄  LOOP PRINCIPAL
---------------------------------------------------------------------
local lastLowTick = 0
RunService.RenderStepped:Connect(function(dt)
    cfg._dtAccumulator += dt
    updateFOV()

    -- decidir se roda nesta iteração (modo leve)
    if cfg.lowPower and cfg._dtAccumulator < 0.1 then return end
    cfg._dtAccumulator = 0

    refreshESP()

    if cfg.aimbot then
        local tgt = getClosest()
        if tgt then
            local camCF = Camera.CFrame
            local dir   = (tgt.Position - camCF.Position).Unit
            local goal  = CFrame.new(camCF.Position, camCF.Position + dir)
            local alpha = 1 - math.pow(1-cfg.smoothness, dt*60)
            Camera.CFrame = camCF:Lerp(goal, alpha)
        end
    end
end)

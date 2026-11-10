-- ==================================================
-- 📋 AA.toki | v31 (HITBOX = como antes / RenderStepped cada frame)
-- ==================================================
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local LocalPlayer = Players.LocalPlayer
local PlayerGui = LocalPlayer:WaitForChild("PlayerGui")

local COLOR_TEXTO = Color3.fromRGB(230, 230, 230)
local COLOR_BARRA = Color3.fromRGB(35, 35, 35)
local TRANSPARENCIA_BARRA = 0.35

local BTN_AQUA_ACTIVA = Color3.fromRGB(120, 240, 210)
local COLOR_CAFE = Color3.fromRGB(75, 48, 22)
local COLOR_BORDE = Color3.fromRGB(255, 255, 255)

local COLOR_FONDO_LISTA = Color3.fromRGB(25, 25, 25)
local COLOR_SELECCIONADO = Color3.fromRGB(60, 60, 60)
local COLOR_ENTRY = Color3.fromRGB(40, 40, 40)
local COLOR_TU = Color3.fromRGB(50, 80, 100)

local TAMANO_HITBOX = 100000
local DISTANCIA_TP_GUARDADO = -500
local ANCHO_BASE = 340
local ALTO_BASE = 352
local MIN_ANCHO = 150
local MIN_ALTO_FRAME = 70
local ID_FONDO = "rbxassetid://79902517665758"

local screenGui = Instance.new("ScreenGui")
screenGui.Name = "AA.toki"
screenGui.Parent = PlayerGui
screenGui.ResetOnSpawn = false
screenGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling

local framePrincipal = Instance.new("Frame")
framePrincipal.Parent = screenGui
framePrincipal.BackgroundColor3 = Color3.fromRGB(15, 15, 15)
framePrincipal.BackgroundTransparency = 0.35
framePrincipal.Size = UDim2.new(0, ANCHO_BASE, 0, ALTO_BASE + 22)
framePrincipal.Position = UDim2.new(0.02, 0, 0.15, 0)
framePrincipal.Active = true
framePrincipal.Draggable = true
framePrincipal.ClipsDescendants = true
framePrincipal:SetAttribute("AnchoOriginal", ANCHO_BASE)
framePrincipal:SetAttribute("AltoOriginal", ALTO_BASE)

local bgImage = Instance.new("ImageLabel")
bgImage.Name = "BackgroundImage"
bgImage.BackgroundTransparency = 1
bgImage.Image = ID_FONDO
bgImage.ImageTransparency = 0.20
bgImage.ScaleType = Enum.ScaleType.Stretch
bgImage.Size = UDim2.new(1, 0, 1, 0)
bgImage.Position = UDim2.new(0, 0, 0, 0)
bgImage.ZIndex = 0
bgImage.Active = false
bgImage.Selectable = false
bgImage.Parent = framePrincipal

local barra = Instance.new("Frame")
barra.Size = UDim2.new(1, 0, 0, 22)
barra.BackgroundColor3 = COLOR_BARRA
barra.BackgroundTransparency = TRANSPARENCIA_BARRA
barra.BorderSizePixel = 1
barra.BorderColor3 = Color3.fromRGB(80, 80, 80)
barra.ZIndex = 10
barra.Parent = framePrincipal

local tituloBarra = Instance.new("TextLabel")
tituloBarra.Size = UDim2.new(0.8, -5, 1, 0)
tituloBarra.Position = UDim2.new(0, 5, 0, 0)
tituloBarra.BackgroundTransparency = 1
tituloBarra.Text = "AA.toki"
tituloBarra.TextColor3 = COLOR_TEXTO
tituloBarra.Font = Enum.Font.GothamBold
tituloBarra.TextSize = 11
tituloBarra.TextXAlignment = Enum.TextXAlignment.Left
tituloBarra.ZIndex = 11
tituloBarra.Parent = barra

local btnMin = Instance.new("TextButton")
btnMin.Size = UDim2.new(0, 22, 0, 22)
btnMin.Position = UDim2.new(1, -22, 0, 0)
btnMin.BackgroundColor3 = Color3.fromRGB(80, 80, 80)
btnMin.BackgroundTransparency = 0.3
btnMin.Text = "-"
btnMin.TextColor3 = Color3.fromRGB(255, 255, 255)
btnMin.Font = Enum.Font.GothamBold
btnMin.TextSize = 13
btnMin.ZIndex = 11
btnMin.Parent = barra

local content = Instance.new("ScrollingFrame")
content.Name = "Content"
content.BackgroundTransparency = 1
content.Position = UDim2.new(0, 0, 0, 22)
content.Size = UDim2.new(1, 0, 1, -22)
content.ClipsDescendants = true
content.BorderSizePixel = 0
content.ScrollBarThickness = 0
content.ScrollBarImageTransparency = 1
content.ScrollingDirection = Enum.ScrollingDirection.Y
content.AutomaticCanvasSize = Enum.AutomaticSize.Y
content.CanvasSize = UDim2.new(0, 0, 0, 0)
content.ZIndex = 2
content.Parent = framePrincipal

local pad = Instance.new("UIPadding")
pad.PaddingTop = UDim.new(0, 8)
pad.PaddingBottom = UDim.new(0, 12)
pad.PaddingLeft = UDim.new(0, 8)
pad.PaddingRight = UDim.new(0, 8)
pad.Parent = content

local layout = Instance.new("UIListLayout")
layout.FillDirection = Enum.FillDirection.Vertical
layout.HorizontalAlignment = Enum.HorizontalAlignment.Center
layout.SortOrder = Enum.SortOrder.LayoutOrder
layout.Padding = UDim.new(0, 6)
layout.Parent = content

local function estilizarBoton(btn)
    btn.BackgroundColor3 = Color3.fromRGB(255, 255, 255)
    btn.BackgroundTransparency = 0.85
    btn.TextColor3 = COLOR_CAFE
    if not btn:FindFirstChildOfClass("UICorner") then
        local c = Instance.new("UICorner")
        c.CornerRadius = UDim.new(0, 4)
        c.Parent = btn
    end
    local stroke = Instance.new("UIStroke")
    stroke.Color = COLOR_BORDE
    stroke.Thickness = 1.5
    stroke.Transparency = 0.05
    stroke.LineJoinMode = Enum.LineJoinMode.Round
    stroke.Parent = btn
    return stroke
end

local function bordeActivo(btn, activo)
    local stroke = btn:FindFirstChildOfClass("UIStroke")
    if not stroke then return end
    if activo then
        stroke.Color = BTN_AQUA_ACTIVA
        stroke.Thickness = 2.5
    else
        stroke.Color = COLOR_BORDE
        stroke.Thickness = 1.5
    end
end

local function filaDosBotones(orden, alto, txtA, txtB, tamTexto)
    tamTexto = tamTexto or 11
    local fila = Instance.new("Frame")
    fila.BackgroundTransparency = 1
    fila.Size = UDim2.new(1, 0, 0, alto)
    fila.LayoutOrder = orden
    fila.Parent = content

    local bA = Instance.new("TextButton")
    bA.Size = UDim2.new(0.5, -3, 1, 0)
    bA.Position = UDim2.new(0, 0, 0, 0)
    bA.Text = txtA
    bA.Font = Enum.Font.GothamBold
    bA.TextSize = tamTexto
    bA.ZIndex = 3
    bA.Parent = fila
    estilizarBoton(bA)

    local bB = Instance.new("TextButton")
    bB.Size = UDim2.new(0.5, -3, 1, 0)
    bB.Position = UDim2.new(0.5, 3, 0, 0)
    bB.Text = txtB
    bB.Font = Enum.Font.GothamBold
    bB.TextSize = tamTexto
    bB.ZIndex = 3
    bB.Parent = fila
    estilizarBoton(bB)

    return bA, bB
end

local function separador(orden, texto)
    local s = Instance.new("TextLabel")
    s.BackgroundTransparency = 1
    s.Size = UDim2.new(1, 0, 0, 14)
    s.LayoutOrder = orden
    s.Text = texto
    s.TextColor3 = Color3.fromRGB(230, 230, 230)
    s.Font = Enum.Font.GothamBold
    s.TextSize = 10
    s.Parent = content
    return s
end

local statusLabel = Instance.new("TextLabel")
statusLabel.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
statusLabel.BackgroundTransparency = 0.5
statusLabel.Size = UDim2.new(1, 0, 0, 24)
statusLabel.LayoutOrder = 1
statusLabel.Text = "Status: Off | Objetivo: Ninguno"
statusLabel.TextColor3 = COLOR_TEXTO
statusLabel.Font = Enum.Font.Gotham
statusLabel.TextSize = 11
statusLabel.Parent = content

local filaManual = Instance.new("Frame")
filaManual.BackgroundTransparency = 1
filaManual.Size = UDim2.new(1, 0, 0, 30)
filaManual.LayoutOrder = 2
filaManual.Parent = content

local txtManual = Instance.new("TextBox")
txtManual.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
txtManual.BackgroundTransparency = 0.3
txtManual.Size = UDim2.new(1, -75, 1, 0)
txtManual.Position = UDim2.new(0, 0, 0, 0)
txtManual.Text = ""
txtManual.PlaceholderText = "Usuario o apodo..."
txtManual.PlaceholderColor3 = Color3.fromRGB(140, 140, 140)
txtManual.TextColor3 = COLOR_TEXTO
txtManual.Font = Enum.Font.Gotham
txtManual.TextSize = 12
txtManual.ClearTextOnFocus = false
txtManual.TextXAlignment = Enum.TextXAlignment.Left
txtManual.Parent = filaManual
Instance.new("UICorner", txtManual).CornerRadius = UDim.new(0, 4)

local btnManualOK = Instance.new("TextButton")
btnManualOK.Size = UDim2.new(0, 70, 1, 0)
btnManualOK.Position = UDim2.new(1, -70, 0, 0)
btnManualOK.Text = "✓ OK"
btnManualOK.Font = Enum.Font.GothamBold
btnManualOK.TextSize = 11
btnManualOK.ZIndex = 3
btnManualOK.Parent = filaManual
estilizarBoton(btnManualOK)

local btnSeleccionarCercano, btnToggleLista = filaDosBotones(3, 32, "🔍 Cercano", "📋 Lista", 10)

local btnToggle = Instance.new("TextButton")
btnToggle.Size = UDim2.new(1, 0, 0, 38)
btnToggle.LayoutOrder = 4
btnToggle.Text = "✅ ACTIVAR"
btnToggle.Font = Enum.Font.GothamBold
btnToggle.TextSize = 13
btnToggle.ZIndex = 3
btnToggle.Parent = content
estilizarBoton(btnToggle)

separador(5, "── 🖤 HITBOX EXPANDER (100K) ──")
local hitboxToggle = Instance.new("TextButton")
hitboxToggle.Size = UDim2.new(1, 0, 0, 34)
hitboxToggle.LayoutOrder = 6
hitboxToggle.Text = "HITBOX: OFF"
hitboxToggle.Font = Enum.Font.GothamBold
hitboxToggle.TextSize = 13
hitboxToggle.ZIndex = 3
hitboxToggle.Parent = content
estilizarBoton(hitboxToggle)

local btnAutopisar, btnAutoagarre = filaDosBotones(7, 34, "⚡ AUTOPISAR: OFF", "🫵 AGARRE: OFF", 9)

local btnTPGuardar, btnDeath = filaDosBotones(8, 34, "💾 TP -500", "💀 DEATH 0.8s", 10)

local btnAbrazo = Instance.new("TextButton")
btnAbrazo.Size = UDim2.new(1, 0, 0, 34)
btnAbrazo.LayoutOrder = 9
btnAbrazo.Text = "🎯 ABRAZO BUG"
btnAbrazo.Font = Enum.Font.GothamBold
btnAbrazo.TextSize = 12
btnAbrazo.ZIndex = 3
btnAbrazo.Parent = content
estilizarBoton(btnAbrazo)

local areaLista = Instance.new("Frame")
areaLista.Name = "AreaLista"
areaLista.Size = UDim2.new(1, 0, 0, 0)
areaLista.LayoutOrder = 10
areaLista.BackgroundTransparency = 1
areaLista.Parent = content

local scrollingFrame = Instance.new("ScrollingFrame")
scrollingFrame.Name = "PlayerList"
scrollingFrame.Size = UDim2.new(1, 0, 1, 0)
scrollingFrame.BackgroundColor3 = COLOR_FONDO_LISTA
scrollingFrame.BackgroundTransparency = 0.5
scrollingFrame.BorderSizePixel = 0
scrollingFrame.CanvasSize = UDim2.new(0, 0, 0, 0)
scrollingFrame.ScrollBarThickness = 4
scrollingFrame.ScrollBarImageColor3 = Color3.fromRGB(120, 120, 120)
scrollingFrame.Parent = areaLista

local uiListLayout = Instance.new("UIListLayout")
uiListLayout.FillDirection = Enum.FillDirection.Vertical
uiListLayout.HorizontalAlignment = Enum.HorizontalAlignment.Center
uiListLayout.SortOrder = Enum.SortOrder.Name
uiListLayout.Padding = UDim.new(0, 6)
uiListLayout.Parent = scrollingFrame

local letraA = Instance.new("TextLabel")
letraA.Size = UDim2.new(1, -28, 0, 16)
letraA.Position = UDim2.new(0, 6, 1, -16)
letraA.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
letraA.BackgroundTransparency = 0.5
letraA.Text = "A"
letraA.TextColor3 = Color3.fromRGB(255, 255, 255)
letraA.Font = Enum.Font.GothamBold
letraA.TextSize = 12
letraA.TextTransparency = 0.3
letraA.TextXAlignment = Enum.TextXAlignment.Left
letraA.ZIndex = 12
letraA.Parent = framePrincipal

local gripResize = Instance.new("TextButton")
gripResize.Size = UDim2.new(0, 18, 0, 18)
gripResize.Position = UDim2.new(1, -18, 1, -18)
gripResize.BackgroundColor3 = Color3.fromRGB(90, 90, 90)
gripResize.BackgroundTransparency = 0.4
gripResize.Text = "◢"
gripResize.TextColor3 = Color3.fromRGB(255, 255, 255)
gripResize.Font = Enum.Font.GothamBold
gripResize.TextSize = 12
gripResize.Active = true
gripResize.ZIndex = 20
gripResize.Parent = framePrincipal
Instance.new("UICorner", gripResize).CornerRadius = UDim.new(0, 4)

local redimensionando = false
local inicioMouse = Vector2.new()
local inicioTamano = Vector2.new()

gripResize.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
        redimensionando = true
        inicioMouse = Vector2.new(input.Position.X, input.Position.Y)
        inicioTamano = Vector2.new(framePrincipal.Size.X.Offset, framePrincipal.Size.Y.Offset)
        framePrincipal.Draggable = false
    end
end)

UserInputService.InputChanged:Connect(function(input)
    if redimensionando and (input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch) then
        local delta = Vector2.new(input.Position.X, input.Position.Y) - inicioMouse
        local nuevoAncho = math.max(MIN_ANCHO, inicioTamano.X + delta.X)
        local nuevoAlto = math.max(MIN_ALTO_FRAME, inicioTamano.Y + delta.Y)
        framePrincipal.Size = UDim2.new(0, nuevoAncho, 0, nuevoAlto)
        framePrincipal:SetAttribute("AnchoOriginal", nuevoAncho)
        framePrincipal:SetAttribute("AltoOriginal", nuevoAlto - 22)
    end
end)

UserInputService.InputEnded:Connect(function(input)
    if (input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch) and redimensionando then
        redimensionando = false
        framePrincipal.Draggable = true
    end
end)

local minimizado = false
btnMin.MouseButton1Click:Connect(function()
    minimizado = not minimizado
    local ancho = framePrincipal:GetAttribute("AnchoOriginal")
    local alto = framePrincipal:GetAttribute("AltoOriginal")
    if minimizado then
        content.Visible = false
        gripResize.Visible = false
        letraA.Visible = false
        framePrincipal.Size = UDim2.new(0, ancho, 0, 22)
        btnMin.Text = "+"
    else
        content.Visible = true
        gripResize.Visible = true
        letraA.Visible = true
        framePrincipal.Size = UDim2.new(0, ancho, 0, alto + 22)
        btnMin.Text = "-"
    end
end)

local jugadorSeleccionado = nil
local teleportEnabled = false
local listaAbierta = false
local originalPositions = {}

local hitboxActivo = false
local hitboxConexion = nil
local hitboxOriginalSize = {}

local lastPosition = nil

local autoMode = false
local targetSaved = nil

local deathActivo = false
local deathConnection = nil
local deathTimerTask = nil

local entradasLista = {}

-- ==================================================
-- ⚡🫵💀 CACHÉ DE REMOTES
-- ==================================================
local palabrasProhibidasAutopisar = {
    "reset","die","death","respawn","kill",
    "pvp","combat","attack","hit","damage",
    "fight","strike","shoot","weapon","melee",
    "health","hp","defense","shield","skill",
    "ability","dodge","block","knock","stun",
    "ruleta","spin","wheel","girar","giro",
    "suerte","lottery","roulette","reward","premio"
}

local EXCLUIR_DEATH = {"stomp","pisar","pisada","step","stepped","suelo","floor","tile","baldosa","footstep"}
local function debeExcluirDeath(nombre)
    local n = string.lower(nombre)
    for _, palabra in ipairs(EXCLUIR_DEATH) do
        if string.find(n, palabra) then return true end
    end
    return false
end

local safeRemotesAutopisar = {}
local safeRemotesAutoagarre = {}
local cachedDeathRemotes = {}

local function escanearRemotes()
    local tAutopisar, tAutoagarre, tDeath = {}, {}, {}
    for _, desc in ipairs(ReplicatedStorage:GetDescendants()) do
        if desc:IsA("RemoteEvent") then
            local rName = desc.Name:lower()
            if not debeExcluirDeath(desc.Name) then table.insert(tDeath, desc) end
            if not (rName:find("reset") or rName:find("die") or rName:find("death") or rName:find("respawn") or rName:find("kill")) then
                if #tAutoagarre < 15 then table.insert(tAutoagarre, desc) end
            end
            local prohibido = false
            for _, p in ipairs(palabrasProhibidasAutopisar) do
                if rName:find(p) then prohibido = true; break end
            end
            if not prohibido and #tAutopisar < 15 then table.insert(tAutopisar, desc) end
        end
    end
    safeRemotesAutopisar = tAutopisar
    safeRemotesAutoagarre = tAutoagarre
    cachedDeathRemotes = tDeath
end

escanearRemotes()
task.spawn(function()
    while true do
        task.wait(2)
        local tDeath = {}
        for _, desc in ipairs(ReplicatedStorage:GetDescendants()) do
            if desc:IsA("RemoteEvent") and not debeExcluirDeath(desc.Name) then table.insert(tDeath, desc) end
        end
        cachedDeathRemotes = tDeath
    end
end)

local autopisarActivo = false
local autopisarThread = nil
local autoagarreActivo = false
local autoagarreThread = nil

btnAutopisar.MouseButton1Click:Connect(function()
    autopisarActivo = not autopisarActivo
    if autopisarActivo then
        btnAutopisar.Text = "⚡ AUTOPISAR: ON"
        bordeActivo(btnAutopisar, true)
        autopisarThread = task.spawn(function()
            while autopisarActivo do
                local remotes = safeRemotesAutopisar
                for i = 1, #remotes do
                    if not autopisarActivo then break end
                    pcall(function() remotes[i]:FireServer(true) end)
                end
                task.wait(0.3)
            end
        end)
    else
        if autopisarThread then task.cancel(autopisarThread); autopisarThread = nil end
        btnAutopisar.Text = "⚡ AUTOPISAR: OFF"
        bordeActivo(btnAutopisar, false)
    end
end)

btnAutoagarre.MouseButton1Click:Connect(function()
    autoagarreActivo = not autoagarreActivo
    if autoagarreActivo then
        btnAutoagarre.Text = "🫵 AGARRE: ON"
        bordeActivo(btnAutoagarre, true)
        autoagarreThread = task.spawn(function()
            while autoagarreActivo do
                local remotes = safeRemotesAutoagarre
                for i = 1, #remotes do
                    if not autoagarreActivo then break end
                    pcall(function() remotes[i]:FireServer(true) end)
                end
                task.wait(0.3)
            end
        end)
    else
        if autoagarreThread then task.cancel(autoagarreThread); autoagarreThread = nil end
        btnAutoagarre.Text = "🫵 AGARRE: OFF"
        bordeActivo(btnAutoagarre, false)
    end
end)

-- ==================================================
-- 💀 DEATH = 0.8s POR CLIC
-- ==================================================
local function apagarDeath()
    deathActivo = false
    if deathConnection then deathConnection:Disconnect(); deathConnection = nil end
    deathTimerTask = nil
    btnDeath.Text = "💀 DEATH 0.8s"
    bordeActivo(btnDeath, false)
end

btnDeath.MouseButton1Click:Connect(function()
    if deathConnection then deathConnection:Disconnect(); deathConnection = nil end
    if deathTimerTask then task.cancel(deathTimerTask); deathTimerTask = nil end
    deathActivo = true
    btnDeath.Text = "⚡ DEATH 0.8s"
    bordeActivo(btnDeath, true)
    deathConnection = RunService.Heartbeat:Connect(function()
        if not deathActivo then return end
        local char = LocalPlayer.Character
        local remotes = cachedDeathRemotes
        for i = 1, #remotes do
            local r = remotes[i]
            if r.Parent then pcall(function() r:FireServer(char, math.huge) end) end
        end
    end)
    deathTimerTask = task.delay(0.8, apagarDeath)
end)

-- ==================================================
-- 🔄 ESTADOS DE BOTONES
-- ==================================================
local function actualizarBotonToggle()
    if teleportEnabled then
        bordeActivo(btnToggle, true)
        btnToggle.Text = "❌ DESACTIVAR"
    else
        bordeActivo(btnToggle, false)
        btnToggle.Text = "✅ ACTIVAR"
    end
end

local function actualizarBotonHitbox()
    if hitboxActivo then
        bordeActivo(hitboxToggle, true)
        hitboxToggle.Text = jugadorSeleccionado and ("HITBOX ON: " .. jugadorSeleccionado.Name:sub(1, 10)) or "HITBOX ON: sin objetivo"
    else
        bordeActivo(hitboxToggle, false)
        hitboxToggle.Text = "HITBOX: OFF"
    end
end

-- ==================================================
-- 🎯 OBJETIVO COMPARTIDO
-- ==================================================
local actualizarListaJugadores

local function refrescarColoresLista()
    for plr, entry in pairs(entradasLista) do
        if entry and entry.Parent then
            if plr == jugadorSeleccionado then
                entry.BackgroundColor3 = COLOR_SELECCIONADO
            elseif plr == LocalPlayer then
                entry.BackgroundColor3 = COLOR_TU
            else
                entry.BackgroundColor3 = COLOR_ENTRY
            end
        end
    end
end

local function establecerObjetivo(plr)
    if plr == jugadorSeleccionado then return end
    local anterior = jugadorSeleccionado
    jugadorSeleccionado = plr
    if hitboxActivo and anterior and anterior ~= plr and hitboxOriginalSize[anterior] then
        if anterior.Character and anterior.Character:FindFirstChild("HumanoidRootPart") then
            anterior.Character.HumanoidRootPart.Size = hitboxOriginalSize[anterior]
        end
        hitboxOriginalSize[anterior] = nil
    end
    refrescarColoresLista()
    actualizarBotonHitbox()
    if plr then
        statusLabel.Text = "Objetivo: " .. plr.Name
        statusLabel.TextColor3 = Color3.fromRGB(180, 255, 230)
    else
        statusLabel.Text = "Status: Off | Objetivo: Ninguno"
        statusLabel.TextColor3 = COLOR_TEXTO
    end
end

function actualizarListaJugadores()
    entradasLista = {}
    for _, child in ipairs(scrollingFrame:GetChildren()) do
        if child:IsA("Frame") then child:Destroy() end
    end
    local listaJugadores = Players:GetPlayers()
    for _, plr in ipairs(listaJugadores) do
        local playerEntry = Instance.new("Frame")
        playerEntry.Size = UDim2.new(1, 0, 0, 38)
        playerEntry.BackgroundColor3 = (jugadorSeleccionado == plr) and COLOR_SELECCIONADO or (plr == LocalPlayer and COLOR_TU or COLOR_ENTRY)
        playerEntry.BackgroundTransparency = 0.2
        playerEntry.BorderSizePixel = 0
        playerEntry.Parent = scrollingFrame
        Instance.new("UICorner", playerEntry).CornerRadius = UDim.new(0, 4)
        entradasLista[plr] = playerEntry
        local nameLabel = Instance.new("TextLabel")
        nameLabel.Size = UDim2.new(1, -15, 1, 0)
        nameLabel.Position = UDim2.new(0, 8, 0, 0)
        nameLabel.BackgroundTransparency = 1
        nameLabel.Text = plr.DisplayName .. " (@" .. plr.Name .. ")" .. (plr == LocalPlayer and "  • TÚ" or "")
        nameLabel.TextColor3 = COLOR_TEXTO
        nameLabel.Font = Enum.Font.GothamSemibold
        nameLabel.TextSize = 13
        nameLabel.TextXAlignment = Enum.TextXAlignment.Left
        nameLabel.Parent = playerEntry
        playerEntry.InputBegan:Connect(function(input)
            if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
                establecerObjetivo(plr)
            end
        end)
    end
    scrollingFrame.CanvasSize = UDim2.new(0, 0, 0, #listaJugadores * 44)
end

Players.PlayerAdded:Connect(actualizarListaJugadores)
Players.PlayerRemoving:Connect(function(plr)
    hitboxOriginalSize[plr] = nil
    entradasLista[plr] = nil
    if plr == jugadorSeleccionado then establecerObjetivo(nil) else actualizarListaJugadores() end
end)

local function obtenerJugadorMasCercano()
    local miCaracter = LocalPlayer.Character
    if not miCaracter or not miCaracter:FindFirstChild("HumanoidRootPart") then return nil end
    local miPos = miCaracter.HumanoidRootPart.Position
    local distanciaMinima = math.huge
    local jugadorCercano = nil
    for _, plr in pairs(Players:GetPlayers()) do
        if plr ~= LocalPlayer and plr.Character and plr.Character:FindFirstChild("HumanoidRootPart") then
            local dist = (miPos - plr.Character.HumanoidRootPart.Position).Magnitude
            if dist < distanciaMinima then distanciaMinima = dist; jugadorCercano = plr end
        end
    end
    return jugadorCercano
end

local function buscarJugadorPorNombre(texto)
    texto = texto:lower():gsub("^%s+", ""):gsub("%s+$", "")
    if texto == "" then return nil end
    for _, plr in pairs(Players:GetPlayers()) do
        if plr.Name:lower() == texto or plr.DisplayName:lower() == texto then return plr end
    end
    for _, plr in pairs(Players:GetPlayers()) do
        if plr.Name:lower():find(texto, 1, true) or plr.DisplayName:lower():find(texto, 1, true) then return plr end
    end
    return nil
end

local function seleccionarManual()
    local encontrado = buscarJugadorPorNombre(txtManual.Text)
    if encontrado then
        establecerObjetivo(encontrado)
        statusLabel.Text = "Objetivo: " .. encontrado.Name .. " (" .. encontrado.DisplayName .. ")"
        statusLabel.TextColor3 = Color3.fromRGB(180, 255, 230)
    else
        statusLabel.Text = "❌ No se encontró: " .. txtManual.Text
        statusLabel.TextColor3 = Color3.fromRGB(255, 120, 120)
    end
end

btnManualOK.MouseButton1Click:Connect(seleccionarManual)
txtManual.FocusLost:Connect(function(enter) if enter then seleccionarManual() end end)

local function teleportarJugadorSeleccionado()
    local char = LocalPlayer.Character
    if not char or not char:FindFirstChild("HumanoidRootPart") then return end
    local myHRP = char.HumanoidRootPart
    local target = jugadorSeleccionado
    if not target or not target.Character or not target.Character:FindFirstChild("HumanoidRootPart") then return end
    local tHRP = target.Character.HumanoidRootPart
    if not originalPositions[target] then originalPositions[target] = tHRP.CFrame end
    local nuevaPosicion = myHRP.CFrame * CFrame.new(0, 0, -3)
    tHRP.CFrame = CFrame.lookAt(nuevaPosicion.Position, myHRP.Position) * CFrame.Angles(0, math.rad(180), 0)
end

local function restaurarPosicionOriginal()
    local target = jugadorSeleccionado
    if target and originalPositions[target] and target.Character and target.Character:FindFirstChild("HumanoidRootPart") then
        target.Character.HumanoidRootPart.CFrame = originalPositions[target]
        originalPositions[target] = nil
    end
end

-- ==================================================
-- 🖤 HITBOX — IGUAL QUE ANTES (RenderStepped cada frame)
-- ==================================================
local function buscarObjetivoHitbox()
    if jugadorSeleccionado and jugadorSeleccionado ~= LocalPlayer then return jugadorSeleccionado end
    return nil
end

local function expandirHitbox()
    local objetivo = buscarObjetivoHitbox()
    if objetivo and objetivo.Character and objetivo.Character:FindFirstChild("HumanoidRootPart") then
        local hrp = objetivo.Character.HumanoidRootPart
        if not hitboxOriginalSize[objetivo] then hitboxOriginalSize[objetivo] = hrp.Size end
        hrp.Size = Vector3.new(TAMANO_HITBOX, TAMANO_HITBOX, TAMANO_HITBOX)
        hrp.Transparency = 1
        hrp.CanCollide = false
    end
end

local function restaurarHitbox(plr)
    if plr and plr.Character and plr.Character:FindFirstChild("HumanoidRootPart") then
        local hrp = plr.Character.HumanoidRootPart
        hrp.Size = hitboxOriginalSize[plr] or Vector3.new(2, 2, 1)
        hrp.Transparency = 1
        hrp.CanCollide = false
    end
end

local function restaurarTodosHitbox()
    for plr, _ in pairs(hitboxOriginalSize) do restaurarHitbox(plr) end
    hitboxOriginalSize = {}
end

hitboxToggle.MouseButton1Click:Connect(function()
    hitboxActivo = not hitboxActivo
    if hitboxActivo then
        local targetPlr = buscarObjetivoHitbox()
        if targetPlr then
            actualizarBotonHitbox()
            hitboxConexion = RunService.RenderStepped:Connect(expandirHitbox)
        else
            hitboxActivo = false
            hitboxToggle.Text = "AVATAR NO ENCONTRADO"
            task.wait(1.2)
            actualizarBotonHitbox()
        end
    else
        actualizarBotonHitbox()
        if hitboxConexion then hitboxConexion:Disconnect(); hitboxConexion = nil end
        restaurarTodosHitbox()
    end
end)

-- ==================================================
-- 💾 TP -500
-- ==================================================
local function guardarYTeletransportar()
    local character = LocalPlayer.Character
    if not character or not character:FindFirstChild("HumanoidRootPart") then
        statusLabel.Text = "❌ Sin personaje"; task.delay(1.5, function() statusLabel.Text = "Status: Listo" end); return
    end
    lastPosition = character.HumanoidRootPart.Position
    statusLabel.Text = "✅ Posición guardada!"
    task.wait(0.3)
    character:PivotTo(CFrame.new(lastPosition + Vector3.new(0, DISTANCIA_TP_GUARDADO, 0)))
    statusLabel.Text = "✅ ¡TP -500!"
    task.delay(1.2, function() statusLabel.Text = "Status: Listo" end)
end

btnTPGuardar.MouseButton1Click:Connect(guardarYTeletransportar)

local function onCharacterAdded(character)
    character:WaitForChild("Humanoid").Died:Connect(function()
        task.wait(0.5)
        if lastPosition then
            local hrp = character:WaitForChild("HumanoidRootPart", 10)
            if hrp then character:PivotTo(CFrame.new(lastPosition)) end
        end
    end)
end
LocalPlayer.CharacterAdded:Connect(onCharacterAdded)
if LocalPlayer.Character then onCharacterAdded(LocalPlayer.Character) end

-- ==================================================
-- 🎯 ABRAZO BUG (noclip optimizado + primer agarre inmediato)
-- ==================================================
local function KillTarget(target)
    if not target or not target.Character then return end
    local tchar = target.Character
    local troot = tchar:FindFirstChild("HumanoidRootPart")
    local thum = tchar:FindFirstChildOfClass("Humanoid")
    local root = LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart")
    if not troot or not thum or not root then return end

    thum.Health = 0
    thum:TakeDamage(999999)
    thum:BreakJoints()
    thum.DisplayDistanceType = Enum.HumanoidDisplayDistanceType.None

    -- Noclip: escanea las partes UNA sola vez (no cada frame)
    local partesNoclip = {}
    local miChar = LocalPlayer.Character
    if miChar then
        for _, p in pairs(miChar:GetDescendants()) do
            if p:IsA("BasePart") then table.insert(partesNoclip, p) end
        end
    end
    local noclip = RunService.Stepped:Connect(function()
        for i = 1, #partesNoclip do
            local p = partesNoclip[i]
            if p and p.Parent then p.CanCollide = false end
        end
    end)

    local bv = Instance.new("BodyVelocity")
    bv.MaxForce = Vector3.new(math.huge, math.huge, math.huge)
    bv.Velocity = Vector3.new(0, -25000, 0)
    bv.Parent = root

    local ba = Instance.new("BodyAngularVelocity")
    ba.MaxTorque = Vector3.new(math.huge, math.huge, math.huge)
    ba.AngularVelocity = Vector3.new(15000, 15000, 15000)
    ba.Parent = root

    local dur = 1.5
    local ini = tick()
    while autoMode and tick() - ini < dur and troot.Parent do
        root.CFrame = troot.CFrame * CFrame.new(0, -2, 0)
        root.AssemblyLinearVelocity = Vector3.new(0, -30000, 0)
        task.wait()
    end

    pcall(function() bv:Destroy() end)
    pcall(function() ba:Destroy() end)
    pcall(function() noclip:Disconnect() end)
end

btnAbrazo.MouseButton1Click:Connect(function()
    if not autoMode then
        local target = obtenerJugadorMasCercano()
        if not target then
            statusLabel.Text = "❌ Nadie cerca"
            task.delay(1.2, function() statusLabel.Text = "Status: Listo" end)
            return
        end
        autoMode = true
        targetSaved = target
        statusLabel.Text = "Fijado: " .. target.Name
        btnAbrazo.Text = "🛑 DETENER"
        bordeActivo(btnAbrazo, true)

        task.spawn(function()
            -- Primer agarre INMEDIATO
            if targetSaved and targetSaved.Character then KillTarget(targetSaved) end
            -- Después repite cada 0.7s
            while autoMode and task.wait(0.7) do
                if targetSaved and targetSaved.Character then
                    KillTarget(targetSaved)
                else
                    autoMode = false
                    break
                end
            end
            autoMode = false
            targetSaved = nil
            btnAbrazo.Text = "🎯 ABRAZO BUG"
            bordeActivo(btnAbrazo, false)
            statusLabel.Text = "Status: Listo"
        end)
    else
        autoMode = false
        targetSaved = nil
        btnAbrazo.Text = "🎯 ABRAZO BUG"
        bordeActivo(btnAbrazo, false)
        statusLabel.Text = "Status: Listo"
    end
end)

-- ==================================================
-- 🖱️ LISTA
-- ==================================================
btnToggleLista.MouseButton1Click:Connect(function()
    listaAbierta = not listaAbierta
    local ancho = framePrincipal.Size.X.Offset
    local altoBase = framePrincipal:GetAttribute("AltoOriginal") or ALTO_BASE
    framePrincipal:SetAttribute("AnchoOriginal", ancho)
    if listaAbierta then
        areaLista.Size = UDim2.new(1, 0, 0, 245)
        framePrincipal.Size = UDim2.new(0, ancho, 0, altoBase + 22 + 245)
        btnToggleLista.Text = "📋 Cerrar"
        actualizarListaJugadores()
    else
        areaLista.Size = UDim2.new(1, 0, 0, 0)
        framePrincipal.Size = UDim2.new(0, ancho, 0, altoBase + 22)
        btnToggleLista.Text = "📋 Lista"
    end
end)

btnSeleccionarCercano.MouseButton1Click:Connect(function()
    local cercano = obtenerJugadorMasCercano()
    if cercano then
        establecerObjetivo(cercano)
    else
        establecerObjetivo(nil)
        statusLabel.Text = "No se encontró jugador"
        statusLabel.TextColor3 = Color3.fromRGB(255, 120, 120)
    end
end)

btnToggle.MouseButton1Click:Connect(function()
    if teleportEnabled then
        teleportEnabled = false
        statusLabel.Text = "Status: Off | Objetivo: " .. (jugadorSeleccionado and jugadorSeleccionado.Name or "Ninguno")
        statusLabel.TextColor3 = Color3.fromRGB(255, 120, 120)
        restaurarPosicionOriginal()
    else
        if not jugadorSeleccionado then
            statusLabel.Text = "¡Selecciona un jugador primero!"
            statusLabel.TextColor3 = Color3.fromRGB(255, 120, 120)
            return
        end
        teleportEnabled = true
        statusLabel.Text = "✅ Activo | Objetivo: " .. jugadorSeleccionado.Name
        statusLabel.TextColor3 = Color3.fromRGB(180, 255, 230)
    end
    actualizarBotonToggle()
end)

task.spawn(function()
    while true do
        if teleportEnabled and jugadorSeleccionado then teleportarJugadorSeleccionado() end
        task.wait(0.1)
    end
end)

task.wait(0.3)
actualizarListaJugadores()

print("✅ AA.toki v31 (hitbox como antes) cargado correctamente")

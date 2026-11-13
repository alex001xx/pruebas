-- ==================================================
-- 📋 AA.toki | v33 (death mode lento + fix pestañas)
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
local COLOR_MULTI = Color3.fromRGB(30, 100, 85) -- resaltado selección múltiple (verde/agua)

local TAMANO_HITBOX = 100000
local DISTANCIA_TP_GUARDADO = -500
local ANCHO_BASE = 340
local ALTO_BASE = 352
local MIN_ANCHO = 150
local MIN_ALTO_FRAME = 70
local ID_FONDO = "rbxassetid://79902517665758"

-- ➕ Pestañas: alto de la barra de tabs y desplazamiento vertical del contenido
local ALTO_TABS = 24
local ALTO_SUPERIOR = 22 + ALTO_TABS -- barra de título + barra de pestañas

local screenGui = Instance.new("ScreenGui")
screenGui.Name = "AA.toki"
screenGui.Parent = PlayerGui
screenGui.ResetOnSpawn = false
screenGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling

local framePrincipal = Instance.new("Frame")
framePrincipal.Parent = screenGui
framePrincipal.BackgroundColor3 = Color3.fromRGB(15, 15, 15)
framePrincipal.BackgroundTransparency = 0.35
framePrincipal.Size = UDim2.new(0, ANCHO_BASE, 0, ALTO_BASE + ALTO_SUPERIOR)
framePrincipal.Position = UDim2.new(0.02, 0, 0.15, 0)
framePrincipal.Active = true
framePrincipal.Draggable = true
framePrincipal.ClipsDescendants = true
framePrincipal:SetAttribute("AnchoOriginal", ANCHO_BASE)
framePrincipal:SetAttribute("AltoOriginal", ALTO_BASE + ALTO_TABS)

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

-- ==================================================
-- ➕ BARRA DE PESTAÑAS (debajo de la barra de título)
-- ==================================================
local content, contentAdicionales -- forward declarations (se crean abajo)
local tabActiva = "principal"

local barraTabs = Instance.new("Frame")
barraTabs.Size = UDim2.new(1, 0, 0, ALTO_TABS)
barraTabs.Position = UDim2.new(0, 0, 0, 22)
barraTabs.BackgroundColor3 = Color3.fromRGB(20, 20, 20)
barraTabs.BackgroundTransparency = 0.4
barraTabs.BorderSizePixel = 0
barraTabs.ZIndex = 10
barraTabs.Parent = framePrincipal

local function crearBotonTab(texto, escalaX)
    local b = Instance.new("TextButton")
    b.Size = UDim2.new(0.5, -3, 1, -4)
    b.Position = UDim2.new(escalaX, 1, 0, 2)
    b.BackgroundColor3 = Color3.fromRGB(60, 60, 60)
    b.BackgroundTransparency = 0.25
    b.Text = texto
    b.TextColor3 = COLOR_TEXTO
    b.Font = Enum.Font.GothamBold
    b.TextSize = 11
    b.ZIndex = 11
    b.Parent = barraTabs
    Instance.new("UICorner", b).CornerRadius = UDim.new(0, 4)
    return b
end

local btnTabPrincipal = crearBotonTab("📋 Principal", 0)
local btnTabAdicionales = crearBotonTab("➕ Adicionales", 0.5)

local function resaltarTab(nombre)
    if nombre == "principal" then
        btnTabPrincipal.BackgroundColor3 = BTN_AQUA_ACTIVA
        btnTabPrincipal.BackgroundTransparency = 0
        btnTabPrincipal.TextColor3 = Color3.fromRGB(15, 15, 15)
        btnTabAdicionales.BackgroundColor3 = Color3.fromRGB(60, 60, 60)
        btnTabAdicionales.BackgroundTransparency = 0.25
        btnTabAdicionales.TextColor3 = COLOR_TEXTO
    else
        btnTabAdicionales.BackgroundColor3 = BTN_AQUA_ACTIVA
        btnTabAdicionales.BackgroundTransparency = 0
        btnTabAdicionales.TextColor3 = Color3.fromRGB(15, 15, 15)
        btnTabPrincipal.BackgroundColor3 = Color3.fromRGB(60, 60, 60)
        btnTabPrincipal.BackgroundTransparency = 0.25
        btnTabPrincipal.TextColor3 = COLOR_TEXTO
    end
end

btnTabPrincipal.MouseButton1Click:Connect(function()
    tabActiva = "principal"
    content.Visible = true
    contentAdicionales.Visible = false
    resaltarTab("principal")
end)

btnTabAdicionales.MouseButton1Click:Connect(function()
    tabActiva = "adicionales"
    content.Visible = false
    contentAdicionales.Visible = true
    resaltarTab("adicionales")
end)

resaltarTab("principal")

content = Instance.new("ScrollingFrame")
content.Name = "Content"
content.BackgroundTransparency = 1
content.Position = UDim2.new(0, 0, 0, ALTO_SUPERIOR)
content.Size = UDim2.new(1, 0, 1, -ALTO_SUPERIOR)
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

-- ☑️ NUEVO: SELECCIÓN MÚLTIPLE (debajo del botón de hitbox)
local btnSeleccionMultiple = Instance.new("TextButton")
btnSeleccionMultiple.Size = UDim2.new(1, 0, 0, 34)
btnSeleccionMultiple.LayoutOrder = 7
btnSeleccionMultiple.Text = "☑️ SELECCIÓN MÚLTIPLE: OFF"
btnSeleccionMultiple.Font = Enum.Font.GothamBold
btnSeleccionMultiple.TextSize = 12
btnSeleccionMultiple.ZIndex = 3
btnSeleccionMultiple.Parent = content
estilizarBoton(btnSeleccionMultiple)

local btnTPGuardar, btnDeath = filaDosBotones(9, 34, "💾 TP -500", "💀 DEATH 0.8s", 10)

local btnAbrazo = Instance.new("TextButton")
btnAbrazo.Size = UDim2.new(1, 0, 0, 34)
btnAbrazo.LayoutOrder = 10
btnAbrazo.Text = "🎯 ABRAZO BUG"
btnAbrazo.Font = Enum.Font.GothamBold
btnAbrazo.TextSize = 12
btnAbrazo.ZIndex = 3
btnAbrazo.Parent = content
estilizarBoton(btnAbrazo)

local areaLista = Instance.new("Frame")
areaLista.Name = "AreaLista"
areaLista.Size = UDim2.new(1, 0, 0, 0)
areaLista.LayoutOrder = 11
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

-- ==================================================
-- ➕ PESTAÑA 2: ADICIONALES
-- ==================================================
contentAdicionales = Instance.new("ScrollingFrame")
contentAdicionales.Name = "Adicionales"
contentAdicionales.BackgroundTransparency = 1
contentAdicionales.Position = UDim2.new(0, 0, 0, ALTO_SUPERIOR)
contentAdicionales.Size = UDim2.new(1, 0, 1, -ALTO_SUPERIOR)
contentAdicionales.ClipsDescendants = true
contentAdicionales.BorderSizePixel = 0
contentAdicionales.ScrollBarThickness = 0
contentAdicionales.ScrollBarImageTransparency = 1
contentAdicionales.ScrollingDirection = Enum.ScrollingDirection.Y
contentAdicionales.AutomaticCanvasSize = Enum.AutomaticSize.Y
contentAdicionales.CanvasSize = UDim2.new(0, 0, 0, 0)
contentAdicionales.ZIndex = 2
contentAdicionales.Visible = false
contentAdicionales.Parent = framePrincipal

local pad2 = Instance.new("UIPadding")
pad2.PaddingTop = UDim.new(0, 8)
pad2.PaddingBottom = UDim.new(0, 12)
pad2.PaddingLeft = UDim.new(0, 8)
pad2.PaddingRight = UDim.new(0, 8)
pad2.Parent = contentAdicionales

local layout2 = Instance.new("UIListLayout")
layout2.FillDirection = Enum.FillDirection.Vertical
layout2.HorizontalAlignment = Enum.HorizontalAlignment.Center
layout2.SortOrder = Enum.SortOrder.LayoutOrder
layout2.Padding = UDim.new(0, 6)
layout2.Parent = contentAdicionales

-- 🛴 SEPARADOR
local sepScooter = Instance.new("TextLabel")
sepScooter.BackgroundTransparency = 1
sepScooter.Size = UDim2.new(1, 0, 0, 14)
sepScooter.LayoutOrder = 1
sepScooter.Text = "── 🛴 SCOOTER ──"
sepScooter.TextColor3 = Color3.fromRGB(230, 230, 230)
sepScooter.Font = Enum.Font.GothamBold
sepScooter.TextSize = 10
sepScooter.Parent = contentAdicionales

-- ℹ️ INFO SCOOTER
local infoScooter = Instance.new("TextLabel")
infoScooter.BackgroundTransparency = 1
infoScooter.Size = UDim2.new(1, 0, 0, 30)
infoScooter.LayoutOrder = 2
infoScooter.Text = "Equipa sola la scooter\nde tu mochila automáticamente"
infoScooter.TextColor3 = COLOR_TEXTO
infoScooter.Font = Enum.Font.Gotham
infoScooter.TextSize = 10
infoScooter.TextWrapped = true
infoScooter.Parent = contentAdicionales

-- 🛴 BOTÓN SCOOTER
local btnScooter = Instance.new("TextButton")
btnScooter.Size = UDim2.new(1, 0, 0, 38)
btnScooter.LayoutOrder = 3
btnScooter.Text = "🛴 SCOOTER: OFF"
btnScooter.Font = Enum.Font.GothamBold
btnScooter.TextSize = 13
btnScooter.ZIndex = 3
btnScooter.Parent = contentAdicionales
estilizarBoton(btnScooter)

-- 📦 VARIABLES SCOOTER
local NOMBRES_SCOOTER = {
    "scoot", "scooter", "scuter", "escooter",
    "patin", "patineta", "monopatin", "hoverboard"
}
local scooterActivo = false
local scooterThread = nil

-- 🔍 BUSCAR SCOOTER
local function buscarScooter()
    local mochila = LocalPlayer:FindFirstChildOfClass("Backpack")
    if not mochila then return nil end
    for _, obj in ipairs(mochila:GetDescendants()) do
        if obj:IsA("Tool") then
            local n = obj.Name:lower()
            for _, clave in ipairs(NOMBRES_SCOOTER) do
                if n:find(clave, 1, true) then return obj end
            end
        end
    end
    return nil
end

-- ⚡ EQUIPAR SCOOTER
local function equiparScooter()
    local char = LocalPlayer.Character
    if not char then return end
    local hum = char:FindFirstChildOfClass("Humanoid")
    if not hum or hum.Health <= 0 then return end
    -- Si ya tiene una equipada, no hacemos nada
    for _, obj in ipairs(char:GetChildren()) do
        if obj:IsA("Tool") then
            local n = obj.Name:lower()
            for _, clave in ipairs(NOMBRES_SCOOTER) do
                if n:find(clave, 1, true) then return end
            end
        end
    end
    -- Buscamos y equipamos
    local tool = buscarScooter()
    if tool then pcall(function() hum:EquipTool(tool) end) end
end

-- 🖱️ CLICK BOTÓN SCOOTER
btnScooter.MouseButton1Click:Connect(function()
    scooterActivo = not scooterActivo
    if scooterActivo then
        btnScooter.Text = "🛴 SCOOTER: ON"
        bordeActivo(btnScooter, true)
        scooterThread = task.spawn(function()
            while scooterActivo do
                equiparScooter()
                task.wait(0.5)
            end
        end)
    else
        if scooterThread then task.cancel(scooterThread); scooterThread = nil end
        btnScooter.Text = "🛴 SCOOTER: OFF"
        bordeActivo(btnScooter, false)
    end
end)

-- ✅ FIN PESTAÑA 2
print("✅ Pestaña Adicionales cargada correctamente")

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
        contentAdicionales.Visible = false
        barraTabs.Visible = false
        gripResize.Visible = false
        letraA.Visible = false
        framePrincipal.Size = UDim2.new(0, ancho, 0, 22)
        btnMin.Text = "+"
    else
        barraTabs.Visible = true
        if tabActiva == "principal" then
            content.Visible = true
            contentAdicionales.Visible = false
        else
            content.Visible = false
            contentAdicionales.Visible = true
        end
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

-- 🎯 ABRAZO BUG (tu código nuevo, tal cual)
local autoMode = false
local targetSaved = nil

local entradasLista = {}

-- ☑️ SELECCIÓN MÚLTIPLE: estado
local seleccionMultipleActiva = false
local seleccionados = {} -- [Player] = true

-- 🔄 AUTO RECONEXIÓN: si un objetivo sale del servidor y vuelve a entrar,
-- se vuelve a seleccionar automáticamente (por nombre de usuario).
local pendingSingleName = nil -- nombre del objetivo único que salió
local reconexionPendienteMulti = {} -- [nombre] = true, para selección múltiple
local UMBRAL_ARRASTRE = 10 -- píxeles; por debajo se considera clic/toca, por encima scroll

local function contarSeleccionados()
    local c = 0
    for _ in pairs(seleccionados) do c = c + 1 end
    return c
end

local function hayObjetivosActivos()
    if seleccionMultipleActiva then
        return next(seleccionados) ~= nil
    end
    return jugadorSeleccionado ~= nil
end

local function iterarObjetivos(fn)
    if seleccionMultipleActiva then
        for plr, _ in pairs(seleccionados) do
            if plr and plr.Parent then fn(plr) end
        end
    elseif jugadorSeleccionado then
        fn(jugadorSeleccionado)
    end
end

-- ==================================================
-- 💀 DEATH MODE — OPTIMIZADO | 0.8s POR CLIC
-- (integrado al botón 💀 DEATH ya existente, sin GUI flotante aparte)
-- ==================================================
-- ⚙️ VELOCIDAD DE REMOTES EN DEATH MODE (ajustable)
-- Más alto = más lento. 0.25 = ~4 disparos por segundo
local INTERVALO_REMOTES = 0.25
local INTERVALO_DIED = 0.20

local deathIsActive = false
local deathCicloActivo = false

-- 📋 FILTROS DE REMOTES
local IncluirPalabras = {
    "death", "died", "kill", "damage", "dmg", "attack", "hit", "combat",
    "pvp", "fight", "strike", "slay", "health", "hp", "hurt", "injure",
    "respawn", "revive", "regen", "heal", "spawn", "reset", "clear",
    "release", "free", "unlock", "escape", "untrap", "liberate",
    "die", "destroy", "break", "drop", "cancel", "stop", "end",
    "knockback", "stun", "paralyze", "push", "fling", "launch", "explode"
}

local ExcluirPalabras = {
    "roulette", "spin", "wheel", "gacha", "ruleta", "footstep", "foot", "step",
    "trail", "particle", "effect", "visual", "emote", "dance", "animation", "pose",
    "shop", "buy", "coin", "gem", "currency", "money", "chat", "message",
    "notification", "popup", "music", "sound", "audio", "ambient", "join", "leave",
    "invite", "friend", "cosmetic", "skin", "hat", "clothing", "quest", "mission",
    "task", "daily"
}

local function DebeIncluirRemote(Nombre)
    local NombreMinus = string.lower(Nombre)
    for _, v in ipairs(ExcluirPalabras) do if NombreMinus:find(v) then return false end end
    for _, v in ipairs(IncluirPalabras) do if NombreMinus:find(v) then return true end end
    return false
end

-- 🌀 IMPULSO
local function ImpulsoSalvaje(Parte)
    if not Parte then return end
    local fuerza = 80000 + math.random(20000, 150000)
    Parte.AssemblyLinearVelocity = Vector3.new(
        math.random(-10, 110),
        math.random(600, 1200),
        -fuerza * 0.5
    )
end

-- 🔓 LIBERAR PERSONAJE
local function RompeAgarresYLibera(Character)
    local Humanoid = Character:FindFirstChildOfClass("Humanoid")
    local RootPart = Character:FindFirstChild("HumanoidRootPart")
    local misPartes = {}

    if RootPart then
        pcall(function() if RootPart:CanSetNetworkOwnership() then RootPart:SetNetworkOwner(LocalPlayer) end end)
        pcall(function() RootPart.Anchored = false; RootPart.CanCollide = true; RootPart.Transparency = 0 end)
        ImpulsoSalvaje(RootPart)
    end

    if Humanoid then
        Humanoid.PlatformStand = false
        Humanoid.Sit = false
        Humanoid.AutoRotate = false
        Humanoid:SetStateEnabled(Enum.HumanoidStateType.Seated, false)
        Humanoid:SetStateEnabled(Enum.HumanoidStateType.Physics, false)
        Humanoid:SetStateEnabled(Enum.HumanoidStateType.Ragdoll, false)
        Humanoid:SetStateEnabled(Enum.HumanoidStateType.FallingDown, false)
        Humanoid:SetStateEnabled(Enum.HumanoidStateType.GettingUp, false)
        Humanoid:SetStateEnabled(Enum.HumanoidStateType.Climbing, false)
        Humanoid:SetStateEnabled(Enum.HumanoidStateType.Dead, true)
    end

    for _, p in ipairs(Character:GetDescendants()) do
        if p:IsA("BasePart") then
            misPartes[p] = true
            pcall(function() p.Transparency = 0; p.Anchored = false; p.CanCollide = true end)
            ImpulsoSalvaje(p)
        end
    end

    local listaDestruir = {
        "WeldConstraint", "Weld", "Motor6D", "ManualWeld", "JointInstance",
        "RodConstraint", "RopeConstraint", "SpringConstraint", "Constraint",
        "AlignPosition", "AlignOrientation", "HingeConstraint", "BallSocketConstraint",
        "PrismaticConstraint", "CylindricalConstraint", "TorsionSpringConstraint",
        "UniversalConstraint", "LineForce", "VectorForce", "Torque", "AngularVelocity",
        "LinearVelocity", "BodyVelocity", "BodyForce", "BodyAngularVelocity",
        "BodyPosition", "BodyGyro", "Attachment", "ParticleEmitter", "Sound"
    }

    for _, clase in ipairs(listaDestruir) do
        for _, obj in ipairs(Character:GetDescendants()) do
            if obj:IsA(clase) and not misPartes[obj] then pcall(function() obj:Destroy() end) end
        end
    end

    for _, obj in ipairs(workspace:GetDescendants()) do
        if obj:IsA("Constraint") or obj:IsA("WeldConstraint") or obj:IsA("Weld") or obj:IsA("JointInstance") then
            local ok0, p0 = pcall(function() return obj.Part0 end)
            local ok1, p1 = pcall(function() return obj.Part1 end)
            if ok0 and ok1 and p0 and p1 and (misPartes[p0] or misPartes[p1]) then
                pcall(function() obj:Destroy() end)
            end
        end
    end
end

-- 💀 VIDA A CERO
local function VidaCeroExtremo(Humanoid)
    local function BucleVida()
        if not deathIsActive or Humanoid.Health <= 0 or not Humanoid.Parent then return end
        pcall(function()
            Humanoid.Health = 0
            Humanoid:TakeDamage(math.huge)
            Humanoid:SetStateEnabled(Enum.HumanoidStateType.Dead, true)
        end)
        task.spawn(BucleVida)
    end
    for _ = 1, 8 do task.spawn(BucleVida) end
end

-- ⚡ REMOTES
local function ReboteRemotesHiper(Character)
    task.spawn(function()
        while deathIsActive do
            pcall(function()
                for _, remote in ipairs(ReplicatedStorage:GetDescendants()) do
                    if remote:IsA("RemoteEvent") and DebeIncluirRemote(remote.Name) then
                        task.spawn(function() pcall(function() remote:FireServer(Character, "death", 0, math.huge, os.time()) end) end)
                    elseif remote:IsA("RemoteFunction") and DebeIncluirRemote(remote.Name) then
                        task.spawn(function() pcall(function() remote:InvokeServer(Character, "death", 0) end) end)
                    end
                end
            end)
            task.wait(INTERVALO_REMOTES)
        end
    end)

    task.spawn(function()
        local Humanoid = Character:FindFirstChildOfClass("Humanoid")
        while deathIsActive and Humanoid and Humanoid.Parent do
            pcall(function() Humanoid.Died:Fire(); Humanoid.HealthChanged:Fire(0) end)
            task.wait(INTERVALO_DIED)
        end
    end)
end

-- 🔁 CICLO PRINCIPAL
local function IniciarCicloDeath()
    if not deathIsActive then return end
    local Character = LocalPlayer.Character
    if not Character then
        pcall(function() LocalPlayer:LoadCharacter() end)
        task.wait(0.1)
        if deathIsActive then IniciarCicloDeath() end
        return
    end

    local Humanoid = Character:FindFirstChildOfClass("Humanoid")
    if not Humanoid then task.wait(0.05); if deathIsActive then IniciarCicloDeath() end return end

    ReboteRemotesHiper(Character)

    local LiberarLoop = RunService.RenderStepped:Connect(function()
        if not deathIsActive or Humanoid.Health <= 0 or not Humanoid.Parent then
            LiberarLoop:Disconnect()
            return
        end
        RompeAgarresYLibera(Character)
    end)

    VidaCeroExtremo(Humanoid)

    local DiedConn = Humanoid.Died:Connect(function()
        DiedConn:Disconnect()
        LiberarLoop:Disconnect()
        btnDeath.Text = "⏳ EN ESPERA..."
        LocalPlayer.CharacterAdded:Connect(function(NuevoPersonaje)
            NuevoPersonaje:WaitForChild("Humanoid", 3)
            task.wait(0.08)
            if deathIsActive then IniciarCicloDeath() end
        end)
    end)
end

-- ⏳ ACTIVADOR 0.8s
local function ActivarDeathPor08s()
    if deathCicloActivo then return end
    deathCicloActivo = true
    deathIsActive = true
    btnDeath.Text = "⚡ ACTIVO 0.8s"
    bordeActivo(btnDeath, true)
    IniciarCicloDeath()

    task.wait(0.8)

    deathIsActive = false
    deathCicloActivo = false
    btnDeath.Text = "💀 DEATH 0.8s"
    bordeActivo(btnDeath, false)
end

btnDeath.MouseButton1Click:Connect(ActivarDeathPor08s)

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
        if seleccionMultipleActiva then
            hitboxToggle.Text = "HITBOX ON x" .. contarSeleccionados()
        else
            hitboxToggle.Text = jugadorSeleccionado and ("HITBOX ON: " .. jugadorSeleccionado.Name:sub(1, 10)) or "HITBOX ON: sin objetivo"
        end
    else
        bordeActivo(hitboxToggle, false)
        hitboxToggle.Text = "HITBOX: OFF"
    end
end

local function actualizarBotonSeleccionMultiple()
    if seleccionMultipleActiva then
        bordeActivo(btnSeleccionMultiple, true)
        btnSeleccionMultiple.Text = "☑️ MÚLTIPLE: ON (" .. contarSeleccionados() .. ")"
    else
        bordeActivo(btnSeleccionMultiple, false)
        btnSeleccionMultiple.Text = "☑️ SELECCIÓN MÚLTIPLE: OFF"
    end
end

-- ==================================================
-- 🎯 OBJETIVO COMPARTIDO
-- ==================================================
local actualizarListaJugadores

local function refrescarColoresLista()
    for plr, entry in pairs(entradasLista) do
        if entry and entry.Parent then
            local nameLabel = entry:FindFirstChildOfClass("TextLabel")
            local baseTexto = plr.DisplayName .. " (@" .. plr.Name .. ")" .. (plr == LocalPlayer and "  • TÚ" or "")
            if seleccionMultipleActiva and seleccionados[plr] then
                entry.BackgroundColor3 = COLOR_MULTI
                if nameLabel then nameLabel.Text = "✓ " .. baseTexto end
            elseif plr == jugadorSeleccionado then
                entry.BackgroundColor3 = COLOR_SELECCIONADO
                if nameLabel then nameLabel.Text = baseTexto end
            elseif plr == LocalPlayer then
                entry.BackgroundColor3 = COLOR_TU
                if nameLabel then nameLabel.Text = baseTexto end
            else
                entry.BackgroundColor3 = COLOR_ENTRY
                if nameLabel then nameLabel.Text = baseTexto end
            end
        end
    end
end

local function establecerObjetivo(plr)
    if plr == jugadorSeleccionado then return end
    local anterior = jugadorSeleccionado
    jugadorSeleccionado = plr
    pendingSingleName = nil -- el usuario cambió/quitó el objetivo, cancelar reconexión automática
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

-- ☑️ Alternar un jugador dentro/ fuera del set de selección múltiple
local function alternarSeleccionMultiple(plr)
    if plr == LocalPlayer then return end -- no te seleccionas a ti mismo
    if seleccionados[plr] then
        seleccionados[plr] = nil
        -- Si el hitbox está activo, restaurar a este jugador
        if hitboxActivo and hitboxOriginalSize[plr] then
            if plr.Character and plr.Character:FindFirstChild("HumanoidRootPart") then
                plr.Character.HumanoidRootPart.Size = hitboxOriginalSize[plr]
            end
            hitboxOriginalSize[plr] = nil
        end
    else
        seleccionados[plr] = true
    end
    refrescarColoresLista()
    actualizarBotonSeleccionMultiple()
    actualizarBotonHitbox()
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
        -- 🖱️ Detección CLIC vs SCROLL: sólo selecciona al SOLTAR y si no hubo arrastre.
        -- Así puedes arrastrar la lista para hacer scroll sin que se seleccione nadie.
        local pressStart = nil
        playerEntry.InputBegan:Connect(function(input)
            if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
                pressStart = Vector2.new(input.Position.X, input.Position.Y)
            end
        end)
        playerEntry.InputChanged:Connect(function(input)
            if pressStart and (input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch) then
                local delta = (Vector2.new(input.Position.X, input.Position.Y) - pressStart).Magnitude
                if delta > UMBRAL_ARRASTRE then pressStart = nil end -- es scroll, cancelar clic
            end
        end)
        playerEntry.InputEnded:Connect(function(input)
            if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
                if pressStart then
                    local delta = (Vector2.new(input.Position.X, input.Position.Y) - pressStart).Magnitude
                    pressStart = nil
                    if delta <= UMBRAL_ARRASTRE then
                        if seleccionMultipleActiva then
                            alternarSeleccionMultiple(plr)
                        else
                            establecerObjetivo(plr)
                        end
                    end
                end
            end
        end)
    end
    refrescarColoresLista() -- aplicar ✓ / color a los ya seleccionados
    scrollingFrame.CanvasSize = UDim2.new(0, 0, 0, #listaJugadores * 44)
end

Players.PlayerAdded:Connect(function(plr)
    actualizarListaJugadores()
    -- 🔄 Si este jugador salió estando seleccionado y vuelve a entrar, re-seleccionarlo
    if pendingSingleName == plr.Name then
        pendingSingleName = nil
        establecerObjetivo(plr)
        statusLabel.Text = "🔄 Re-seleccionado: " .. plr.Name
        statusLabel.TextColor3 = Color3.fromRGB(180, 255, 230)
    elseif reconexionPendienteMulti[plr.Name] then
        reconexionPendienteMulti[plr.Name] = nil
        if plr ~= LocalPlayer then
            seleccionados[plr] = true
            actualizarBotonSeleccionMultiple()
            actualizarBotonHitbox()
            refrescarColoresLista()
            statusLabel.Text = "🔄 Re-añadido: " .. plr.Name .. " (total " .. contarSeleccionados() .. ")"
            statusLabel.TextColor3 = Color3.fromRGB(180, 255, 230)
        end
    end
end)

Players.PlayerRemoving:Connect(function(plr)
    hitboxOriginalSize[plr] = nil
    entradasLista[plr] = nil
    originalPositions[plr] = nil
    if plr == jugadorSeleccionado then
        -- 🔄 El objetivo único salió: marcar para re-seleccionar al volver (script sigue encendido)
        pendingSingleName = plr.Name
        jugadorSeleccionado = nil
        actualizarListaJugadores()
        refrescarColoresLista()
        actualizarBotonHitbox()
        actualizarBotonSeleccionMultiple()
        statusLabel.Text = "⏳ " .. plr.Name .. " salió — esperando reconexión..."
        statusLabel.TextColor3 = Color3.fromRGB(255, 220, 120)
    else
        if seleccionados[plr] then
            reconexionPendienteMulti[plr.Name] = true
            seleccionados[plr] = nil
            actualizarBotonSeleccionMultiple()
            actualizarBotonHitbox()
        end
        actualizarListaJugadores()
    end
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
        if seleccionMultipleActiva then
            alternarSeleccionMultiple(encontrado)
            statusLabel.Text = (seleccionados[encontrado] and "✓ Añadido: " or "➖ Quitado: ") .. encontrado.Name .. "  (total " .. contarSeleccionados() .. ")"
            statusLabel.TextColor3 = Color3.fromRGB(180, 255, 230)
        else
            establecerObjetivo(encontrado)
            statusLabel.Text = "Objetivo: " .. encontrado.Name .. " (" .. encontrado.DisplayName .. ")"
            statusLabel.TextColor3 = Color3.fromRGB(180, 255, 230)
        end
    else
        statusLabel.Text = "❌ No se encontró: " .. txtManual.Text
        statusLabel.TextColor3 = Color3.fromRGB(255, 120, 120)
    end
end

btnManualOK.MouseButton1Click:Connect(seleccionarManual)
txtManual.FocusLost:Connect(function(enter) if enter then seleccionarManual() end end)

-- ☑️ Teleport visual: aplica a objetivo único O a todos los seleccionados
local function teleportarJugadorSeleccionado()
    local char = LocalPlayer.Character
    if not char or not char:FindFirstChild("HumanoidRootPart") then return end
    local myHRP = char.HumanoidRootPart
    iterarObjetivos(function(target)
        if target and target.Character and target.Character:FindFirstChild("HumanoidRootPart") then
            local tHRP = target.Character.HumanoidRootPart
            if not originalPositions[target] then originalPositions[target] = tHRP.CFrame end
            local nuevaPosicion = myHRP.CFrame * CFrame.new(0, 0, -3)
            tHRP.CFrame = CFrame.lookAt(nuevaPosicion.Position, myHRP.Position) * CFrame.Angles(0, math.rad(180), 0)
        end
    end)
end

local function restaurarPosicionOriginal()
    for target, cf in pairs(originalPositions) do
        if target and target.Character and target.Character:FindFirstChild("HumanoidRootPart") then
            target.Character.HumanoidRootPart.CFrame = cf
        end
    end
    originalPositions = {}
end

-- ==================================================
-- 🖤 HITBOX (soporta selección múltiple)
-- ==================================================
local function buscarObjetivoHitbox()
    if jugadorSeleccionado and jugadorSeleccionado ~= LocalPlayer then return jugadorSeleccionado end
    return nil
end

local function expandirHitbox()
    iterarObjetivos(function(objetivo)
        if objetivo and objetivo.Character and objetivo.Character:FindFirstChild("HumanoidRootPart") then
            local hrp = objetivo.Character.HumanoidRootPart
            if not hitboxOriginalSize[objetivo] then hitboxOriginalSize[objetivo] = hrp.Size end
            hrp.Size = Vector3.new(TAMANO_HITBOX, TAMANO_HITBOX, TAMANO_HITBOX)
            hrp.Transparency = 1
            hrp.CanCollide = false
        end
    end)
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
        if not hayObjetivosActivos() then
            hitboxActivo = false
            hitboxToggle.Text = seleccionMultipleActiva and "SELECCIONA ALGUIEN" or "AVATAR NO ENCONTRADO"
            task.wait(1.2)
            actualizarBotonHitbox()
            return
        end
        actualizarBotonHitbox()
        hitboxConexion = RunService.RenderStepped:Connect(expandirHitbox)
    else
        actualizarBotonHitbox()
        if hitboxConexion then hitboxConexion:Disconnect(); hitboxConexion = nil end
        restaurarTodosHitbox()
    end
end)

-- ☑️ Botón SELECCIÓN MÚLTIPLE
btnSeleccionMultiple.MouseButton1Click:Connect(function()
    seleccionMultipleActiva = not seleccionMultipleActiva
    if seleccionMultipleActiva then
        -- Lleva el objetivo único actual al set múltiple
        if jugadorSeleccionado and jugadorSeleccionado ~= LocalPlayer then
            seleccionados[jugadorSeleccionado] = true
        end
        bordeActivo(btnSeleccionMultiple, true)
        statusLabel.Text = "☑️ Modo múltiple ON — toca jugadores en la lista"
        statusLabel.TextColor3 = Color3.fromRGB(180, 255, 230)
    else
        -- Al desactivar: restaura hitbox de los seleccionados y limpia el set
        for plr, _ in pairs(seleccionados) do
            if hitboxActivo and hitboxOriginalSize[plr] then
                restaurarHitbox(plr)
                hitboxOriginalSize[plr] = nil
            end
        end
        seleccionados = {}
        reconexionPendienteMulti = {}
        bordeActivo(btnSeleccionMultiple, false)
        statusLabel.Text = "Status: Listo"
        statusLabel.TextColor3 = COLOR_TEXTO
    end
    actualizarBotonSeleccionMultiple()
    actualizarBotonHitbox()
    refrescarColoresLista()
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
-- 🎯 ABRAZO BUG — TU CÓDIGO NUEVO (tal cual, sin su GUI)
-- ==================================================
local function KillTarget(target)
    if not target or not target.Character then return end
    local tchar = target.Character
    local troot = tchar:FindFirstChild("HumanoidRootPart")
    local thum = tchar:FindFirstChildOfClass("Humanoid")
    local root = LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart")
    if not troot or not thum or not root then return end

    -- Daño total
    thum.Health = 0
    thum:TakeDamage(999999)
    thum:BreakJoints()
    thum.DisplayDistanceType = Enum.HumanoidDisplayDistanceType.None

    -- Noclip temporal (tu personaje)
    local noclip = RunService.Stepped:Connect(function()
        local c = LocalPlayer.Character
        if c then
            for _, p in pairs(c:GetDescendants()) do
                if p:IsA("BasePart") then p.CanCollide = false end
            end
        end
    end)

    -- Fuerza combinada (en TU personaje)
    local bv = Instance.new("BodyVelocity")
    bv.MaxForce = Vector3.new(math.huge, math.huge, math.huge)
    bv.Velocity = Vector3.new(0, -25000, 0)
    bv.Parent = root

    local ba = Instance.new("BodyAngularVelocity")
    ba.MaxTorque = Vector3.new(math.huge, math.huge, math.huge)
    ba.AngularVelocity = Vector3.new(15000, 15000, 15000)
    ba.Parent = root

    -- Pegarse al objetivo (debajo) y lanzarse
    local dur = 1.5
    local ini = tick()
    while autoMode and tick() - ini < dur and troot.Parent do
        root.CFrame = troot.CFrame * CFrame.new(0, -2, 0)
        root.AssemblyLinearVelocity = Vector3.new(0, -30000, 0)
        task.wait()
    end

    -- Limpiar
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
        if seleccionMultipleActiva then
            alternarSeleccionMultiple(cercano)
            statusLabel.Text = (seleccionados[cercano] and "✓ Añadido cercano: " or "➖ Quitado: ") .. cercano.Name .. "  (total " .. contarSeleccionados() .. ")"
            statusLabel.TextColor3 = Color3.fromRGB(180, 255, 230)
        else
            establecerObjetivo(cercano)
        end
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
        if not hayObjetivosActivos() then
            statusLabel.Text = "¡Selecciona un jugador primero!"
            statusLabel.TextColor3 = Color3.fromRGB(255, 120, 120)
            return
        end
        teleportEnabled = true
        if seleccionMultipleActiva then
            statusLabel.Text = "✅ TP visual ON x" .. contarSeleccionados()
        else
            statusLabel.Text = "✅ Activo | Objetivo: " .. jugadorSeleccionado.Name
        end
        statusLabel.TextColor3 = Color3.fromRGB(180, 255, 230)
    end
    actualizarBotonToggle()
end)

task.spawn(function()
    while true do
        if teleportEnabled and hayObjetivosActivos() then teleportarJugadorSeleccionado() end
        task.wait(0.1)
    end
end)

task.wait(0.3)
actualizarListaJugadores()

print("✅ AA.toki v33 (death mode lento + fix pestañas) cargado correctamente")
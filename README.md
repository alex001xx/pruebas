-- ==================================================
-- 📋 AA.toki | v30.1 (botones pequeños p/móvil 411px + FIX PESTAÑAS)
-- - GUI reducida al mismo tamaño compacto que la otra versión.
-- - FIX PESTAÑAS: "content" ahora es forward-declared y NO se redeclara
--   con "local" después (antes, al tocar una pestaña, content era nil y
--   el script fallaba silenciosamente, así que no cambiaba de pestaña).
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
local COLOR_MULTI = Color3.fromRGB(30, 100, 85)

local TAMANO_HITBOX = 100000
local DISTANCIA_TP_GUARDADO = -500

-- ==================================================
-- 📏 DIMENSIONES COMPACTAS (móvil 411px)
-- ==================================================
local ANCHO_BASE = 210
local ALTO_BASE = 195          -- un poco más alto por la fila AUTOPISAR/AGARRE extra
local MIN_ANCHO = 110
local MIN_ALTO_FRAME = 55
local ALTO_TITULO = 17         -- barra superior (antes 22)
local ALTO_TABS = 15           -- barra de pestañas (antes 24)
local ALTO_SUPERIOR = ALTO_TITULO + ALTO_TABS
local ALTO_ENTRADA_LISTA = 20  -- antes 38
local ALTO_LISTA_ABIERTA = 140 -- antes 245
local RADIO_ESQUINA = 3        -- antes 4

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
barra.Size = UDim2.new(1, 0, 0, ALTO_TITULO)
barra.BackgroundColor3 = COLOR_BARRA
barra.BackgroundTransparency = TRANSPARENCIA_BARRA
barra.BorderSizePixel = 1
barra.BorderColor3 = Color3.fromRGB(80, 80, 80)
barra.ZIndex = 10
barra.Parent = framePrincipal

local tituloBarra = Instance.new("TextLabel")
tituloBarra.Size = UDim2.new(1, -(ALTO_TITULO + 8), 1, 0)
tituloBarra.Position = UDim2.new(0, 4, 0, 0)
tituloBarra.BackgroundTransparency = 1
tituloBarra.Text = "AA.toki"
tituloBarra.TextColor3 = COLOR_TEXTO
tituloBarra.Font = Enum.Font.GothamBold
tituloBarra.TextSize = 7
tituloBarra.TextXAlignment = Enum.TextXAlignment.Left
tituloBarra.ZIndex = 11
tituloBarra.Parent = barra

local btnMin = Instance.new("TextButton")
btnMin.Size = UDim2.new(0, ALTO_TITULO, 0, ALTO_TITULO)
btnMin.Position = UDim2.new(1, -ALTO_TITULO, 0, 0)
btnMin.BackgroundColor3 = Color3.fromRGB(80, 80, 80)
btnMin.BackgroundTransparency = 0.3
btnMin.Text = "-"
btnMin.TextColor3 = Color3.fromRGB(255, 255, 255)
btnMin.Font = Enum.Font.GothamBold
btnMin.TextSize = 9
btnMin.ZIndex = 11
btnMin.Parent = barra

-- ==================================================
-- ➕ BARRA DE PESTAÑAS (FIX: content Y contentAdicionales forward-declared)
-- ==================================================
local content, contentAdicionales -- forward declarations (se crean abajo)
local tabActiva = "principal"

local barraTabs = Instance.new("Frame")
barraTabs.Size = UDim2.new(1, 0, 0, ALTO_TABS)
barraTabs.Position = UDim2.new(0, 0, 0, ALTO_TITULO)
barraTabs.BackgroundColor3 = Color3.fromRGB(20, 20, 20)
barraTabs.BackgroundTransparency = 0.4
barraTabs.BorderSizePixel = 0
barraTabs.ZIndex = 10
barraTabs.Parent = framePrincipal

local function crearBotonTab(texto, escalaX)
	local b = Instance.new("TextButton")
	b.Size = UDim2.new(0.5, -3, 1, -3)
	b.Position = UDim2.new(escalaX, 1, 0, 1)
	b.BackgroundColor3 = Color3.fromRGB(60, 60, 60)
	b.BackgroundTransparency = 0.25
	b.Text = texto
	b.TextColor3 = COLOR_TEXTO
	b.Font = Enum.Font.GothamBold
	b.TextSize = 7
	b.ZIndex = 11
	b.Parent = barraTabs
	Instance.new("UICorner", b).CornerRadius = UDim.new(0, RADIO_ESQUINA)
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

-- FIX: SIN "local" — se asigna a la variable forward-declarada arriba.
-- (Con "local" aquí se creaba una variable NUEVA y los handlers de pestañas
--  seguían referenciando la vieja, que seguía en nil.)
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
pad.PaddingTop = UDim.new(0, 3)
pad.PaddingBottom = UDim.new(0, 5)
pad.PaddingLeft = UDim.new(0, 3)
pad.PaddingRight = UDim.new(0, 3)
pad.Parent = content

local layout = Instance.new("UIListLayout")
layout.FillDirection = Enum.FillDirection.Vertical
layout.HorizontalAlignment = Enum.HorizontalAlignment.Center
layout.SortOrder = Enum.SortOrder.LayoutOrder
layout.Padding = UDim.new(0, 2)
layout.Parent = content

local function estilizarBoton(btn)
	btn.BackgroundColor3 = Color3.fromRGB(255, 255, 255)
	btn.BackgroundTransparency = 0.85
	btn.TextColor3 = COLOR_CAFE
	if not btn:FindFirstChildOfClass("UICorner") then
		local c = Instance.new("UICorner")
		c.CornerRadius = UDim.new(0, RADIO_ESQUINA)
		c.Parent = btn
	end
	local stroke = Instance.new("UIStroke")
	stroke.Color = COLOR_BORDE
	stroke.Thickness = 1
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
		stroke.Thickness = 1.5
	else
		stroke.Color = COLOR_BORDE
		stroke.Thickness = 1
	end
end

local function filaDosBotones(orden, alto, txtA, txtB, tamTexto)
	tamTexto = tamTexto or 8
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
	s.Size = UDim2.new(1, 0, 0, 9)
	s.LayoutOrder = orden
	s.Text = texto
	s.TextColor3 = Color3.fromRGB(230, 230, 230)
	s.Font = Enum.Font.GothamBold
	s.TextSize = 6
	s.Parent = content
	return s
end

local statusLabel = Instance.new("TextLabel")
statusLabel.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
statusLabel.BackgroundTransparency = 0.5
statusLabel.Size = UDim2.new(1, 0, 0, 14)
statusLabel.LayoutOrder = 1
statusLabel.Text = "Status: Off | Objetivo: Ninguno"
statusLabel.TextColor3 = COLOR_TEXTO
statusLabel.Font = Enum.Font.Gotham
statusLabel.TextSize = 7
statusLabel.Parent = content

local filaManual = Instance.new("Frame")
filaManual.BackgroundTransparency = 1
filaManual.Size = UDim2.new(1, 0, 0, 18)
filaManual.LayoutOrder = 2
filaManual.Parent = content

local txtManual = Instance.new("TextBox")
txtManual.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
txtManual.BackgroundTransparency = 0.3
txtManual.Size = UDim2.new(1, -52, 1, 0)
txtManual.Position = UDim2.new(0, 0, 0, 0)
txtManual.Text = ""
txtManual.PlaceholderText = "Usuario o apodo..."
txtManual.PlaceholderColor3 = Color3.fromRGB(140, 140, 140)
txtManual.TextColor3 = COLOR_TEXTO
txtManual.Font = Enum.Font.Gotham
txtManual.TextSize = 7
txtManual.ClearTextOnFocus = false
txtManual.TextXAlignment = Enum.TextXAlignment.Left
txtManual.Parent = filaManual
Instance.new("UICorner", txtManual).CornerRadius = UDim.new(0, RADIO_ESQUINA)

local btnManualOK = Instance.new("TextButton")
btnManualOK.Size = UDim2.new(0, 48, 1, 0)
btnManualOK.Position = UDim2.new(1, -48, 0, 0)
btnManualOK.Text = "✓ OK"
btnManualOK.Font = Enum.Font.GothamBold
btnManualOK.TextSize = 8
btnManualOK.ZIndex = 3
btnManualOK.Parent = filaManual
estilizarBoton(btnManualOK)

local btnSeleccionarCercano, btnToggleLista = filaDosBotones(3, 18, "🔍 Cercano", "📋 Lista", 7)

local btnToggle = Instance.new("TextButton")
btnToggle.Size = UDim2.new(1, 0, 0, 20)
btnToggle.LayoutOrder = 4
btnToggle.Text = "✅ ACTIVAR"
btnToggle.Font = Enum.Font.GothamBold
btnToggle.TextSize = 8
btnToggle.ZIndex = 3
btnToggle.Parent = content
estilizarBoton(btnToggle)

separador(5, "── 🖤 HITBOX EXPANDER (100K) ──")

local hitboxToggle = Instance.new("TextButton")
hitboxToggle.Size = UDim2.new(1, 0, 0, 20)
hitboxToggle.LayoutOrder = 6
hitboxToggle.Text = "HITBOX: OFF"
hitboxToggle.Font = Enum.Font.GothamBold
hitboxToggle.TextSize = 8
hitboxToggle.ZIndex = 3
hitboxToggle.Parent = content
estilizarBoton(hitboxToggle)

local btnSeleccionMultiple = Instance.new("TextButton")
btnSeleccionMultiple.Size = UDim2.new(1, 0, 0, 20)
btnSeleccionMultiple.LayoutOrder = 7
btnSeleccionMultiple.Text = "☑️ SELECCIÓN MÚLTIPLE: OFF"
btnSeleccionMultiple.Font = Enum.Font.GothamBold
btnSeleccionMultiple.TextSize = 7
btnSeleccionMultiple.ZIndex = 3
btnSeleccionMultiple.Parent = content
estilizarBoton(btnSeleccionMultiple)

local btnAutopisar, btnAutoagarre = filaDosBotones(8, 20, "⚡ AUTOPISAR: OFF", "🫵 AGARRE: OFF", 7)

local btnTPGuardar, btnDeath = filaDosBotones(9, 20, "💾 TP -500", "💀 DEATH 0.8s", 7)

local btnAbrazo = Instance.new("TextButton")
btnAbrazo.Size = UDim2.new(1, 0, 0, 20)
btnAbrazo.LayoutOrder = 10
btnAbrazo.Text = "🎯 ABRAZO BUG"
btnAbrazo.Font = Enum.Font.GothamBold
btnAbrazo.TextSize = 7
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
scrollingFrame.ScrollBarThickness = 2
scrollingFrame.ScrollBarImageColor3 = Color3.fromRGB(120, 120, 120)
scrollingFrame.Parent = areaLista

local uiListLayout = Instance.new("UIListLayout")
uiListLayout.FillDirection = Enum.FillDirection.Vertical
uiListLayout.HorizontalAlignment = Enum.HorizontalAlignment.Center
uiListLayout.SortOrder = Enum.SortOrder.Name
uiListLayout.Padding = UDim.new(0, 2)
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
pad2.PaddingTop = UDim.new(0, 3)
pad2.PaddingBottom = UDim.new(0, 5)
pad2.PaddingLeft = UDim.new(0, 3)
pad2.PaddingRight = UDim.new(0, 3)
pad2.Parent = contentAdicionales

local layout2 = Instance.new("UIListLayout")
layout2.FillDirection = Enum.FillDirection.Vertical
layout2.HorizontalAlignment = Enum.HorizontalAlignment.Center
layout2.SortOrder = Enum.SortOrder.LayoutOrder
layout2.Padding = UDim.new(0, 2)
layout2.Parent = contentAdicionales

local sepScooter = Instance.new("TextLabel")
sepScooter.BackgroundTransparency = 1
sepScooter.Size = UDim2.new(1, 0, 0, 9)
sepScooter.LayoutOrder = 1
sepScooter.Text = "── 🛴 SCOOTER ──"
sepScooter.TextColor3 = Color3.fromRGB(230, 230, 230)
sepScooter.Font = Enum.Font.GothamBold
sepScooter.TextSize = 6
sepScooter.Parent = contentAdicionales

local infoScooter = Instance.new("TextLabel")
infoScooter.BackgroundTransparency = 1
infoScooter.Size = UDim2.new(1, 0, 0, 18)
infoScooter.LayoutOrder = 2
infoScooter.Text = "Equipa sola la scooter\nde tu mochila automáticamente"
infoScooter.TextColor3 = COLOR_TEXTO
infoScooter.Font = Enum.Font.Gotham
infoScooter.TextSize = 6
infoScooter.TextWrapped = true
infoScooter.Parent = contentAdicionales

local btnScooter = Instance.new("TextButton")
btnScooter.Size = UDim2.new(1, 0, 0, 20)
btnScooter.LayoutOrder = 3
btnScooter.Text = "🛴 SCOOTER: OFF"
btnScooter.Font = Enum.Font.GothamBold
btnScooter.TextSize = 8
btnScooter.ZIndex = 3
btnScooter.Parent = contentAdicionales
estilizarBoton(btnScooter)

local NOMBRES_SCOOTER = {
	"scoot", "scooter", "scuter", "escooter",
	"patin", "patineta", "monopatin", "hoverboard"
}
local scooterActivo = false
local scooterThread = nil

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

local function equiparScooter()
	local char = LocalPlayer.Character
	if not char then return end
	local hum = char:FindFirstChildOfClass("Humanoid")
	if not hum or hum.Health <= 0 then return end
	for _, obj in ipairs(char:GetChildren()) do
		if obj:IsA("Tool") then
			local n = obj.Name:lower()
			for _, clave in ipairs(NOMBRES_SCOOTER) do
				if n:find(clave, 1, true) then return end
			end
		end
	end
	local tool = buscarScooter()
	if tool then pcall(function() hum:EquipTool(tool) end) end
end

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

print("✅ Pestaña Adicionales cargada correctamente")

local letraA = Instance.new("TextLabel")
letraA.Size = UDim2.new(1, -18, 0, 10)
letraA.Position = UDim2.new(0, 3, 1, -11)
letraA.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
letraA.BackgroundTransparency = 0.5
letraA.Text = "A"
letraA.TextColor3 = Color3.fromRGB(255, 255, 255)
letraA.Font = Enum.Font.GothamBold
letraA.TextSize = 7
letraA.TextTransparency = 0.3
letraA.TextXAlignment = Enum.TextXAlignment.Left
letraA.ZIndex = 12
letraA.Parent = framePrincipal

local gripResize = Instance.new("TextButton")
gripResize.Size = UDim2.new(0, 12, 0, 12)
gripResize.Position = UDim2.new(1, -12, 1, -12)
gripResize.BackgroundColor3 = Color3.fromRGB(90, 90, 90)
gripResize.BackgroundTransparency = 0.4
gripResize.Text = "◢"
gripResize.TextColor3 = Color3.fromRGB(255, 255, 255)
gripResize.Font = Enum.Font.GothamBold
gripResize.TextSize = 7
gripResize.Active = true
gripResize.ZIndex = 20
gripResize.Parent = framePrincipal
Instance.new("UICorner", gripResize).CornerRadius = UDim.new(0, RADIO_ESQUINA)

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
		framePrincipal:SetAttribute("AltoOriginal", nuevoAlto - ALTO_TITULO)
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
		framePrincipal.Size = UDim2.new(0, ancho, 0, ALTO_TITULO)
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
		framePrincipal.Size = UDim2.new(0, ancho, 0, alto + ALTO_TITULO)
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

local seleccionMultipleActiva = false
local seleccionados = {}

local pendingSingleName = nil
local reconexionPendienteMulti = {}
local UMBRAL_ARRASTRE = 10

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
	pendingSingleName = nil
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

local function alternarSeleccionMultiple(plr)
	if plr == LocalPlayer then return end
	if seleccionados[plr] then
		seleccionados[plr] = nil
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
		playerEntry.Size = UDim2.new(1, 0, 0, ALTO_ENTRADA_LISTA)
		playerEntry.BackgroundColor3 = (jugadorSeleccionado == plr) and COLOR_SELECCIONADO or (plr == LocalPlayer and COLOR_TU or COLOR_ENTRY)
		playerEntry.BackgroundTransparency = 0.2
		playerEntry.BorderSizePixel = 0
		playerEntry.Parent = scrollingFrame
		Instance.new("UICorner", playerEntry).CornerRadius = UDim.new(0, RADIO_ESQUINA)
		entradasLista[plr] = playerEntry

		local nameLabel = Instance.new("TextLabel")
		nameLabel.Size = UDim2.new(1, -8, 1, 0)
		nameLabel.Position = UDim2.new(0, 4, 0, 0)
		nameLabel.BackgroundTransparency = 1
		nameLabel.Text = plr.DisplayName .. " (@" .. plr.Name .. ")" .. (plr == LocalPlayer and "  • TÚ" or "")
		nameLabel.TextColor3 = COLOR_TEXTO
		nameLabel.Font = Enum.Font.GothamSemibold
		nameLabel.TextSize = 7
		nameLabel.TextXAlignment = Enum.TextXAlignment.Left
		nameLabel.Parent = playerEntry

		local pressStart = nil
		playerEntry.InputBegan:Connect(function(input)
			if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
				pressStart = Vector2.new(input.Position.X, input.Position.Y)
			end
		end)
		playerEntry.InputChanged:Connect(function(input)
			if pressStart and (input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch) then
				local delta = (Vector2.new(input.Position.X, input.Position.Y) - pressStart).Magnitude
				if delta > UMBRAL_ARRASTRE then pressStart = nil end
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
	refrescarColoresLista()
	scrollingFrame.CanvasSize = UDim2.new(0, 0, 0, #listaJugadores * (ALTO_ENTRADA_LISTA + 4))
end

Players.PlayerAdded:Connect(function(plr)
	actualizarListaJugadores()
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
-- 🖤 HITBOX
-- ==================================================
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

btnSeleccionMultiple.MouseButton1Click:Connect(function()
	seleccionMultipleActiva = not seleccionMultipleActiva
	if seleccionMultipleActiva then
		if jugadorSeleccionado and jugadorSeleccionado ~= LocalPlayer then
			seleccionados[jugadorSeleccionado] = true
		end
		bordeActivo(btnSeleccionMultiple, true)
		statusLabel.Text = "☑️ Modo múltiple ON — toca jugadores en la lista"
		statusLabel.TextColor3 = Color3.fromRGB(180, 255, 230)
	else
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
-- 🎯 ABRAZO BUG
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

	local noclip = RunService.Stepped:Connect(function()
		local c = LocalPlayer.Character
		if c then
			for _, p in pairs(c:GetDescendants()) do
				if p:IsA("BasePart") then p.CanCollide = false end
			end
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
		areaLista.Size = UDim2.new(1, 0, 0, ALTO_LISTA_ABIERTA)
		framePrincipal.Size = UDim2.new(0, ancho, 0, altoBase + ALTO_TITULO + ALTO_LISTA_ABIERTA)
		btnToggleLista.Text = "📋 Cerrar"
		actualizarListaJugadores()
	else
		areaLista.Size = UDim2.new(1, 0, 0, 0)
		framePrincipal.Size = UDim2.new(0, ancho, 0, altoBase + ALTO_TITULO)
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

print("✅ AA.toki v30.1 (botones pequeños + FIX pestañas) cargado correctamente")
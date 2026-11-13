
-- ==================================================
-- 📋 AA.toki | v38 (UNA SOLA PESTAÑA, versión limpia)
-- Eliminado: pestaña 2/scooter, AUTOPISAR, AUTOAGARRE,
--            HITBOX expander y DEATH mode.
-- Conservado: TP visual (objetivo/múltiple), selección manual +
--             cercano + lista, multi-select, TP -500, ABRAZO BUG.
-- Tamaños compactos para móvil 411px.
-- ==================================================
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
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

local DISTANCIA_TP_GUARDADO = -500

-- ==================================================
-- 📏 DIMENSIONES COMPACTAS (móvil 411px)
-- ==================================================
local ANCHO_BASE = 210
local ALTO_BASE = 155
local MIN_ANCHO = 110
local MIN_ALTO_FRAME = 50
local ALTO_TITULO = 17
local ALTO_ENTRADA_LISTA = 20
local ALTO_LISTA_ABIERTA = 140
local RADIO_ESQUINA = 3

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
framePrincipal.Size = UDim2.new(0, ANCHO_BASE, 0, ALTO_BASE + ALTO_TITULO)
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

-- ==================================================
-- 🏷️ BARRA SUPERIOR
-- ==================================================
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
-- 📄 CONTENIDO (una sola pestaña)
-- ==================================================
local content = Instance.new("ScrollingFrame")
content.Name = "Content"
content.BackgroundTransparency = 1
content.Position = UDim2.new(0, 0, 0, ALTO_TITULO)
content.Size = UDim2.new(1, 0, 1, -ALTO_TITULO)
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

local btnSeleccionMultiple = Instance.new("TextButton")
btnSeleccionMultiple.Size = UDim2.new(1, 0, 0, 20)
btnSeleccionMultiple.LayoutOrder = 5
btnSeleccionMultiple.Text = "☑️ SELECCIÓN MÚLTIPLE: OFF"
btnSeleccionMultiple.Font = Enum.Font.GothamBold
btnSeleccionMultiple.TextSize = 7
btnSeleccionMultiple.ZIndex = 3
btnSeleccionMultiple.Parent = content
estilizarBoton(btnSeleccionMultiple)

local btnTPGuardar = Instance.new("TextButton")
btnTPGuardar.Size = UDim2.new(1, 0, 0, 20)
btnTPGuardar.LayoutOrder = 6
btnTPGuardar.Text = "💾 TP -500"
btnTPGuardar.Font = Enum.Font.GothamBold
btnTPGuardar.TextSize = 8
btnTPGuardar.ZIndex = 3
btnTPGuardar.Parent = content
estilizarBoton(btnTPGuardar)

local btnAbrazo = Instance.new("TextButton")
btnAbrazo.Size = UDim2.new(1, 0, 0, 20)
btnAbrazo.LayoutOrder = 7
btnAbrazo.Text = "🎯 ABRAZO BUG"
btnAbrazo.Font = Enum.Font.GothamBold
btnAbrazo.TextSize = 7
btnAbrazo.ZIndex = 3
btnAbrazo.Parent = content
estilizarBoton(btnAbrazo)

local areaLista = Instance.new("Frame")
areaLista.Name = "AreaLista"
areaLista.Size = UDim2.new(1, 0, 0, 0)
areaLista.LayoutOrder = 8
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
		gripResize.Visible = false
		letraA.Visible = false
		framePrincipal.Size = UDim2.new(0, ancho, 0, ALTO_TITULO)
		btnMin.Text = "+"
	else
		content.Visible = true
		gripResize.Visible = true
		letraA.Visible = true
		framePrincipal.Size = UDim2.new(0, ancho, 0, alto + ALTO_TITULO)
		btnMin.Text = "-"
	end
end)

-- ==================================================
-- 🎯 ESTADO Y SELECCIÓN DE JUGADORES
-- ==================================================
local jugadorSeleccionado = nil
local teleportEnabled = false
local listaAbierta = false
local originalPositions = {}
local lastPosition = nil
local autoMode = false
local targetSaved = nil
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
	if seleccionMultipleActiva then return next(seleccionados) ~= nil end
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

local function actualizarBotonToggle()
	if teleportEnabled then
		bordeActivo(btnToggle, true)
		btnToggle.Text = "❌ DESACTIVAR"
	else
		bordeActivo(btnToggle, false)
		btnToggle.Text = "✅ ACTIVAR"
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
	jugadorSeleccionado = plr
	pendingSingleName = nil
	refrescarColoresLista()
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
	else
		seleccionados[plr] = true
	end
	refrescarColoresLista()
	actualizarBotonSeleccionMultiple()
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
			refrescarColoresLista()
			statusLabel.Text = "🔄 Re-añadido: " .. plr.Name .. " (total " .. contarSeleccionados() .. ")"
			statusLabel.TextColor3 = Color3.fromRGB(180, 255, 230)
		end
	end
end)

Players.PlayerRemoving:Connect(function(plr)
	entradasLista[plr] = nil
	originalPositions[plr] = nil
	if plr == jugadorSeleccionado then
		pendingSingleName = plr.Name
		jugadorSeleccionado = nil
		actualizarListaJugadores()
		refrescarColoresLista()
		actualizarBotonSeleccionMultiple()
		statusLabel.Text = "⏳ " .. plr.Name .. " salió — esperando reconexión..."
		statusLabel.TextColor3 = Color3.fromRGB(255, 220, 120)
	else
		if seleccionados[plr] then
			reconexionPendienteMulti[plr.Name] = true
			seleccionados[plr] = nil
			actualizarBotonSeleccionMultiple()
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

-- ==================================================
-- ✨ TP VISUAL (objetivo único o múltiples)
-- ==================================================
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

-- ☑️ Botón SELECCIÓN MÚLTIPLE
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
		seleccionados = {}
		reconexionPendienteMulti = {}
		bordeActivo(btnSeleccionMultiple, false)
		statusLabel.Text = "Status: Listo"
		statusLabel.TextColor3 = COLOR_TEXTO
	end
	actualizarBotonSeleccionMultiple()
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

print("✅ AA.toki v38 (una pestaña, limpio, botones pequeños) cargado correctamente")
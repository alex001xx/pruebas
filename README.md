-- ==================================================
-- 📋 AA.toki | v36 (botones aún más pequeños + traductor FIX)
-- - UI reducida para móvil 411px (ancho 210, botones 18-20, texto 7-8)
-- - Traductor corregido: detección por LocalizationService (Roblox + Sistema)
--   con fallback a Player.LocaleId y a "en". NUNCA más el script se rompe
--   por acceder a una propiedad que no existe (todo va en pcall).
-- - Botón 🌐 en la barra superior para cambiar de idioma manualmente:
--   ES → EN → PT → JA → ZH (cambia TODO el texto al instante).
-- - Completadas las claves que faltaban en las tablas de JA/ZH/PT
--   (separadores, 💾 TP -500, 🎯 ABRAZO BUG, 🛴 SCOOTER, 💀 DEATH, etc.).
-- ==================================================
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local LocalizationService = game:GetService("LocalizationService")

local LocalPlayer = Players.LocalPlayer
local PlayerGui = LocalPlayer:WaitForChild("PlayerGui")

-- ==================================================
-- 🌐 IDIOMA — DETECCIÓN ROBUSTA + OVERRIDE MANUAL
-- ==================================================
-- Si quieres forzar un idioma SIN depender de la detección automática,
-- pon aquí: "es", "en", "pt", "ja" o "zh". En nil = automático.
local IDIOMA_FORZADO = nil

local function normalizarCodigo(s)
	if type(s) ~= "string" or s == "" then return nil end
	s = s:lower():gsub("-", "_"):gsub("%s+", "")
	if #s < 2 then return nil end
	return s:sub(1, 2)
end

local function detectarIdiomaAuto()
	local ok, id, cod
	-- 1) Idioma elegido DENTRO de Roblox (el más fiable)
	ok, id = pcall(function() return LocalizationService.RobloxLocaleId end)
	if ok then cod = normalizarCodigo(id); if cod then return cod end end
	-- 2) Idioma del sistema operativo / dispositivo
	ok, id = pcall(function() return LocalizationService.SystemLocaleId end)
	if ok then cod = normalizarCodigo(id); if cod then return cod end end
	-- 3) Fallback clásico (algunos ejecutores lo devuelven nil; pcall evita el crash)
	ok, id = pcall(function() return LocalPlayer.LocaleId end)
	if ok then cod = normalizarCodigo(id); if cod then return cod end end
	return "en"
end

local IDIOMAS_SOPORTADOS = { es = true, en = true, ja = true, zh = true, pt = true }
local IDIOMA = IDIOMA_FORZADO or detectarIdiomaAuto()
if not IDIOMAS_SOPORTADOS[IDIOMA] then IDIOMA = "en" end
print("🌐 AA.toki — idioma detectado:", IDIOMA)

-- ==================================================
-- 📚 TABLAS DE TRADUCCIÓN (COMPLETADAS)
-- ==================================================
local TR = {
	en = {
		["📋 Principal"] = "📋 Main",
		["➕ Adicionales"] = "➕ Extras",
		["Status: Off | Objetivo: Ninguno"] = "Status: Off | Target: None",
		["Usuario o apodo..."] = "Username or nickname...",
		["🔍 Cercano"] = "🔍 Nearest",
		["📋 Lista"] = "📋 List",
		["📋 Cerrar"] = "📋 Close",
		["✅ ACTIVAR"] = "✅ ENABLE",
		["❌ DESACTIVAR"] = "❌ DISABLE",
		["HITBOX: OFF"] = "HITBOX: OFF",
		["HITBOX ON: sin objetivo"] = "HITBOX ON: no target",
		["AVATAR NO ENCONTRADO"] = "AVATAR NOT FOUND",
		["SELECCIONA ALGUIEN"] = "SELECT SOMEONE",
		["☑️ SELECCIÓN MÚLTIPLE: OFF"] = "☑️ MULTI-SELECT: OFF",
		["☑️ MÚLTIPLE: ON ("] = "☑️ MULTI: ON (",
		["💀 DEATH 0.8s"] = "💀 DEATH 0.8s",
		["⚡ ACTIVO 0.8s"] = "⚡ ACTIVE 0.8s",
		["⏳ ESPERA..."] = "⏳ WAIT...",
		["⏳ EN ESPERA..."] = "⏳ WAITING...",
		["🛑 DETENER"] = "🛑 STOP",
		["Equipa sola la scooter\nde tu mochila automáticamente"] = "Auto-equips the scooter\nfrom your backpack",
		["🛴 SCOOTER: OFF"] = "🛴 SCOOTER: OFF",
		["🛴 SCOOTER: ON"] = "🛴 SCOOTER: ON",
		["❌ Sin personaje"] = "❌ No character",
		["✅ Posición guardada!"] = "✅ Position saved!",
		["✅ ¡TP -500!"] = "✅ TP -500!",
		["Status: Listo"] = "Status: Ready",
		["❌ Nadie cerca"] = "❌ Nobody near",
		["Fijado: "] = "Locked: ",
		["¡Selecciona un jugador primero!"] = "Select a player first!",
		["No se encontró jugador"] = "Player not found",
		["❌ No se encontró: "] = "❌ Not found: ",
		["Objetivo: "] = "Target: ",
		["✅ Activo | Objetivo: "] = "✅ Active | Target: ",
		["Status: Off | Objetivo: "] = "Status: Off | Target: ",
		[" salió — esperando reconexión..."] = " left — waiting for reconnection...",
		["🔄 Re-seleccionado: "] = "🔄 Re-selected: ",
		["🔄 Re-añadido: "] = "🔄 Re-added: ",
		["✓ Añadido: "] = "✓ Added: ",
		["➖ Quitado: "] = "➖ Removed: ",
		[" (total "] = " (total ",
		["☑️ Modo múltiple ON — toca jugadores en la lista"] = "☑️ Multi mode ON — tap players in list",
		["HITBOX ON x"] = "HITBOX ON x",
		["✅ TP visual ON x"] = "✅ Visual TP ON x",
		[" • TÚ"] = " • YOU",
		["Ninguno"] = "None",
		["✓ Añadido cercano: "] = "✓ Nearest added: ",
		["── 🖤 HITBOX EXPANDER (100K) ──"] = "── 🖤 HITBOX EXPAND (100K) ──",
		["── 🛴 SCOOTER ──"] = "── 🛴 SCOOTER ──",
		["💾 TP -500"] = "💾 TP -500",
		["🎯 ABRAZO BUG"] = "🎯 HUG BUG",
	},
	ja = {
		["📋 Principal"] = "📋 メイン",
		["➕ Adicionales"] = "➕ 追加",
		["Status: Off | Objetivo: Ninguno"] = "状態: オフ | 対象: なし",
		["Usuario o apodo..."] = "ユーザー名 or ニックネーム...",
		["🔍 Cercano"] = "🔍 最寄り",
		["📋 Lista"] = "📋 リスト",
		["📋 Cerrar"] = "📋 閉じる",
		["✅ ACTIVAR"] = "✅ 有効化",
		["❌ DESACTIVAR"] = "❌ 無効化",
		["HITBOX: OFF"] = "ヒットボックス: オフ",
		["HITBOX ON: sin objetivo"] = "ヒットボックスON: 対象なし",
		["AVATAR NO ENCONTRADO"] = "アバターが見つかりません",
		["SELECCIONA ALGUIEN"] = "誰か選んでください",
		["☑️ SELECCIÓN MÚLTIPLE: OFF"] = "☑️ 複数選択: オフ",
		["☑️ MÚLTIPLE: ON ("] = "☑️ 複数: ON (",
		["💀 DEATH 0.8s"] = "💀 デス 0.8秒",
		["⚡ ACTIVO 0.8s"] = "⚡ アクティブ 0.8秒",
		["⏳ ESPERA..."] = "⏳ 待機中...",
		["⏳ EN ESPERA..."] = "⏳ 待機中...",
		["🛑 DETENER"] = "🛑 停止",
		["Equipa sola la scooter\nde tu mochila automáticamente"] = "リュックからスクーターを\n自動装備します",
		["🛴 SCOOTER: OFF"] = "🛴 スクーター: オフ",
		["🛴 SCOOTER: ON"] = "🛴 スクーター: オン",
		["❌ Sin personaje"] = "❌ キャラクターなし",
		["✅ Posición guardada!"] = "✅ 位置を保存!",
		["✅ ¡TP -500!"] = "✅ TP -500!",
		["Status: Listo"] = "状態: 準備完了",
		["❌ Nadie cerca"] = "❌ 近くに誰もいません",
		["Fijado: "] = "固定: ",
		["¡Selecciona un jugador primero!"] = "先にプレイヤーを選んでください!",
		["No se encontró jugador"] = "プレイヤーが見つかりません",
		["❌ No se encontró: "] = "❌ 見つかりません: ",
		["Objetivo: "] = "対象: ",
		["✅ Activo | Objetivo: "] = "✅ アクティブ | 対象: ",
		["Status: Off | Objetivo: "] = "状態: オフ | 対象: ",
		[" salió — esperando reconexión..."] = " が退出 — 再接続待ち...",
		["🔄 Re-seleccionado: "] = "🔄 再選択: ",
		["🔄 Re-añadido: "] = "🔄 再追加: ",
		["✓ Añadido: "] = "✓ 追加: ",
		["➖ Quitado: "] = "➖ 削除: ",
		[" (total "] = " (合計 ",
		["☑️ Modo múltiple ON — toca jugadores en la lista"] = "☑️ 複数モードON — リストをタップ",
		["HITBOX ON x"] = "ヒットボックスON x",
		["✅ TP visual ON x"] = "✅ 視覚TP ON x",
		[" • TÚ"] = " • あなた",
		["Ninguno"] = "なし",
		["✓ Añadido cercano: "] = "✓ 最寄りを追加: ",
		["── 🖤 HITBOX EXPANDER (100K) ──"] = "── 🖤 ヒットボックス拡張 (100K) ──",
		["── 🛴 SCOOTER ──"] = "── 🛴 スクーター ──",
		["💾 TP -500"] = "💾 TP -500",
		["🎯 ABRAZO BUG"] = "🎯 ハグバグ",
	},
	zh = {
		["📋 Principal"] = "📋 主页",
		["➕ Adicionales"] = "➕ 附加",
		["Status: Off | Objetivo: Ninguno"] = "状态: 关 | 目标: 无",
		["Usuario o apodo..."] = "用户名或昵称...",
		["🔍 Cercano"] = "🔍 最近",
		["📋 Lista"] = "📋 列表",
		["📋 Cerrar"] = "📋 关闭",
		["✅ ACTIVAR"] = "✅ 启用",
		["❌ DESACTIVAR"] = "❌ 禁用",
		["HITBOX: OFF"] = "命中框: 关",
		["HITBOX ON: sin objetivo"] = "命中框开: 无目标",
		["AVATAR NO ENCONTRADO"] = "未找到角色",
		["SELECCIONA ALGUIEN"] = "请选择某人",
		["☑️ SELECCIÓN MÚLTIPLE: OFF"] = "☑️ 多选: 关",
		["☑️ MÚLTIPLE: ON ("] = "☑️ 多选: 开 (",
		["💀 DEATH 0.8s"] = "💀 死亡 0.8秒",
		["⚡ ACTIVO 0.8s"] = "⚡ 激活 0.8秒",
		["⏳ ESPERA..."] = "⏳ 等待中...",
		["⏳ EN ESPERA..."] = "⏳ 等待中...",
		["🛑 DETENER"] = "🛑 停止",
		["Equipa sola la scooter\nde tu mochila automáticamente"] = "自动从背包装备\n滑板车",
		["🛴 SCOOTER: OFF"] = "🛴 滑板车: 关",
		["🛴 SCOOTER: ON"] = "🛴 滑板车: 开",
		["❌ Sin personaje"] = "❌ 无角色",
		["✅ Posición guardada!"] = "✅ 位置已保存!",
		["✅ ¡TP -500!"] = "✅ TP -500!",
		["Status: Listo"] = "状态: 就绪",
		["❌ Nadie cerca"] = "❌ 附近没人",
		["Fijado: "] = "锁定: ",
		["¡Selecciona un jugador primero!"] = "请先选择玩家!",
		["No se encontró jugador"] = "未找到玩家",
		["❌ No se encontró: "] = "❌ 未找到: ",
		["Objetivo: "] = "目标: ",
		["✅ Activo | Objetivo: "] = "✅ 已激活 | 目标: ",
		["Status: Off | Objetivo: "] = "状态: 关 | 目标: ",
		[" salió — esperando reconexión..."] = " 已离开 — 等待重连...",
		["🔄 Re-seleccionado: "] = "🔄 重新选中: ",
		["🔄 Re-añadido: "] = "🔄 重新添加: ",
		["✓ Añadido: "] = "✓ 已添加: ",
		["➖ Quitado: "] = "➖ 已移除: ",
		[" (total "] = " (共 ",
		["☑️ Modo múltiple ON — toca jugadores en la lista"] = "☑️ 多选模式开 — 点列表玩家",
		["HITBOX ON x"] = "命中框开 x",
		["✅ TP visual ON x"] = "✅ 视觉传送开 x",
		[" • TÚ"] = " • 你",
		["Ninguno"] = "无",
		["✓ Añadido cercano: "] = "✓ 已添加最近: ",
		["── 🖤 HITBOX EXPANDER (100K) ──"] = "── 🖤 命中框扩大 (100K) ──",
		["── 🛴 SCOOTER ──"] = "── 🛴 滑板车 ──",
		["💾 TP -500"] = "💾 TP -500",
		["🎯 ABRAZO BUG"] = "🎯 拥抱BUG",
	},
	pt = {
		["📋 Principal"] = "📋 Principal",
		["➕ Adicionales"] = "➕ Extras",
		["Status: Off | Objetivo: Ninguno"] = "Status: Off | Alvo: Nenhum",
		["Usuario o apodo..."] = "Usuário ou apelido...",
		["🔍 Cercano"] = "🔍 Próximo",
		["📋 Lista"] = "📋 Lista",
		["📋 Cerrar"] = "📋 Fechar",
		["✅ ACTIVAR"] = "✅ ATIVAR",
		["❌ DESACTIVAR"] = "❌ DESATIVAR",
		["HITBOX: OFF"] = "HITBOX: OFF",
		["HITBOX ON: sin objetivo"] = "HITBOX ON: sem alvo",
		["AVATAR NO ENCONTRADO"] = "AVATAR NÃO ENCONTRADO",
		["SELECCIONA ALGUIEN"] = "SELECIONE ALGUÉM",
		["☑️ SELECCIÓN MÚLTIPLE: OFF"] = "☑️ SELEÇÃO MÚLTIPLA: OFF",
		["☑️ MÚLTIPLE: ON ("] = "☑️ MÚLTIPLO: ON (",
		["💀 DEATH 0.8s"] = "💀 DEATH 0.8s",
		["⚡ ACTIVO 0.8s"] = "⚡ ATIVO 0.8s",
		["⏳ ESPERA..."] = "⏳ ESPERA...",
		["⏳ EN ESPERA..."] = "⏳ AGUARDANDO...",
		["🛑 DETENER"] = "🛑 PARAR",
		["Equipa sola la scooter\nde tu mochila automáticamente"] = "Equipa a scooter sozinha\ndo inventário automaticamente",
		["🛴 SCOOTER: OFF"] = "🛴 SCOOTER: OFF",
		["🛴 SCOOTER: ON"] = "🛴 SCOOTER: ON",
		["❌ Sin personaje"] = "❌ Sem personagem",
		["✅ Posición guardada!"] = "✅ Posição salva!",
		["✅ ¡TP -500!"] = "✅ TP -500!",
		["Status: Listo"] = "Status: Pronto",
		["❌ Nadie cerca"] = "❌ Ninguém por perto",
		["Fijado: "] = "Fixado: ",
		["¡Selecciona un jugador primero!"] = "Selecione um jogador primeiro!",
		["No se encontró jugador"] = "Jogador não encontrado",
		["❌ No se encontró: "] = "❌ Não encontrado: ",
		["Objetivo: "] = "Alvo: ",
		["✅ Activo | Objetivo: "] = "✅ Ativo | Alvo: ",
		["Status: Off | Objetivo: "] = "Status: Off | Alvo: ",
		[" salió — esperando reconexión..."] = " saiu — aguardando reconexão...",
		["🔄 Re-seleccionado: "] = "🔄 Re-selecionado: ",
		["🔄 Re-añadido: "] = "🔄 Re-adicionado: ",
		["✓ Añadido: "] = "✓ Adicionado: ",
		["➖ Quitado: "] = "➖ Removido: ",
		[" (total "] = " (total ",
		["☑️ Modo múltiple ON — toca jugadores en la lista"] = "☑️ Modo múltiplo ON — toque na lista",
		["HITBOX ON x"] = "HITBOX ON x",
		["✅ TP visual ON x"] = "✅ TP visual ON x",
		[" • TÚ"] = " • VOCÊ",
		["Ninguno"] = "Nenhum",
		["✓ Añadido cercano: "] = "✓ Adicionado próximo: ",
		["── 🖤 HITBOX EXPANDER (100K) ──"] = "── 🖤 HITBOX EXPANDER (100K) ──",
		["── 🛴 SCOOTER ──"] = "── 🛴 SCOOTER ──",
		["💾 TP -500"] = "💾 TP -500",
		["🎯 ABRAZO BUG"] = "🎯 ABRAÇO BUG",
	},
}
-- Español = idioma base (las claves ya están en español); si IDIOMA=="es", T() devuelve la clave.

local function T(texto)
	local t = TR[IDIOMA]
	if t and t[texto] ~= nil then return t[texto] end
	return texto
end

-- ==================================================
-- 🎨 COLORES
-- ==================================================
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
-- 📏 DIMENSIONES AÚN MÁS PEQUEÑAS (móvil 411px)
-- ==================================================
local ANCHO_BASE = 210          -- antes 230 (v35) / 300 (original)
local ALTO_BASE = 175           -- antes 200 / 270
local MIN_ANCHO = 110
local MIN_ALTO_FRAME = 55
local ALTO_TITULO = 17          -- barra superior
local ALTO_TABS = 15            -- barra de pestañas
local ALTO_SUPERIOR = ALTO_TITULO + ALTO_TABS
local ALTO_ENTRADA_LISTA = 20   -- antes 22 / 30
local ALTO_LISTA_ABIERTA = 120  -- antes 140 / 200
local RADIO_ESQUINA = 2         -- antes 3 / 4

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

-- ==================================================
-- 🏷️ BARRA SUPERIOR (título + 🌐 idioma + minimizar)
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
tituloBarra.Size = UDim2.new(1, -(ALTO_TITULO * 2 + 8), 1, 0)
tituloBarra.Position = UDim2.new(0, 4, 0, 0)
tituloBarra.BackgroundTransparency = 1
tituloBarra.Text = "AA.toki"
tituloBarra.TextColor3 = COLOR_TEXTO
tituloBarra.Font = Enum.Font.GothamBold
tituloBarra.TextSize = 7
tituloBarra.TextXAlignment = Enum.TextXAlignment.Left
tituloBarra.ZIndex = 11
tituloBarra.Parent = barra

-- 🌐 Botón de idioma (cambia ES→EN→PT→JA→ZH al tocarlo)
local btnIdioma = Instance.new("TextButton")
btnIdioma.Size = UDim2.new(0, ALTO_TITULO, 0, ALTO_TITULO)
btnIdioma.Position = UDim2.new(1, -(ALTO_TITULO * 2), 0, 0)
btnIdioma.BackgroundColor3 = Color3.fromRGB(80, 80, 80)
btnIdioma.BackgroundTransparency = 0.3
btnIdioma.Text = "🌐"
btnIdioma.TextColor3 = Color3.fromRGB(255, 255, 255)
btnIdioma.Font = Enum.Font.GothamBold
btnIdioma.TextSize = 8
btnIdioma.ZIndex = 11
btnIdioma.Parent = barra

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
-- ➕ BARRA DE PESTAÑAS
-- ==================================================
local content, contentAdicionales
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

local NOMBRES_SCOOTER = { "scoot", "scooter", "scuter", "escooter", "patin", "patineta", "monopatin", "hoverboard" }
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
		btnScooter.Text = T("🛴 SCOOTER: ON")
		bordeActivo(btnScooter, true)
		scooterThread = task.spawn(function()
			while scooterActivo do equiparScooter(); task.wait(0.5) end
		end)
	else
		if scooterThread then task.cancel(scooterThread); scooterThread = nil end
		btnScooter.Text = T("🛴 SCOOTER: OFF")
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
			content.Visible = true; contentAdicionales.Visible = false
		else
			content.Visible = false; contentAdicionales.Visible = true
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
local entradasLista = {}
local seleccionMultipleActiva = false
local seleccionados = {}
local pendingSingleName = nil
local reconexionPendienteMulti = {}
local UMBRAL_ARRASTRE = 10
local INTERVALO_REMOTES = 0.25
local INTERVALO_DIED = 0.20
local deathIsActive = false
local deathCicloActivo = false

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

-- ==================================================
-- 💀 DEATH MODE
-- ==================================================
local IncluirPalabras = {
	"death", "died", "kill", "damage", "dmg", "attack", "hit", "combat", "pvp", "fight", "strike", "slay",
	"health", "hp", "hurt", "injure", "respawn", "revive", "regen", "heal", "spawn", "reset", "clear",
	"release", "free", "unlock", "escape", "untrap", "liberate", "die", "destroy", "break", "drop",
	"cancel", "stop", "end", "knockback", "stun", "paralyze", "push", "fling", "launch", "explode"
}
local ExcluirPalabras = {
	"roulette", "spin", "wheel", "gacha", "ruleta", "footstep", "foot", "step", "trail", "particle",
	"effect", "visual", "emote", "dance", "animation", "pose", "shop", "buy", "coin", "gem", "currency",
	"money", "chat", "message", "notification", "popup", "music", "sound", "audio", "ambient", "join",
	"leave", "invite", "friend", "cosmetic", "skin", "hat", "clothing", "quest", "mission", "task", "daily"
}

local function DebeIncluirRemote(Nombre)
	local NombreMinus = string.lower(Nombre)
	for _, v in ipairs(ExcluirPalabras) do if NombreMinus:find(v) then return false end end
	for _, v in ipairs(IncluirPalabras) do if NombreMinus:find(v) then return true end end
	return false
end

local function ImpulsoSalvaje(Parte)
	if not Parte then return end
	local fuerza = 80000 + math.random(20000, 150000)
	Parte.AssemblyLinearVelocity = Vector3.new(math.random(-10, 110), math.random(600, 1200), -fuerza * 0.5)
end

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
		"WeldConstraint", "Weld", "Motor6D", "ManualWeld", "JointInstance", "RodConstraint", "RopeConstraint",
		"SpringConstraint", "Constraint", "AlignPosition", "AlignOrientation", "HingeConstraint",
		"BallSocketConstraint", "PrismaticConstraint", "CylindricalConstraint", "TorsionSpringConstraint",
		"UniversalConstraint", "LineForce", "VectorForce", "Torque", "AngularVelocity", "LinearVelocity",
		"BodyVelocity", "BodyForce", "BodyAngularVelocity", "BodyPosition", "BodyGyro", "Attachment",
		"ParticleEmitter", "Sound"
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
	if not Humanoid then
		task.wait(0.05); if deathIsActive then IniciarCicloDeath() end
		return
	end

	ReboteRemotesHiper(Character)

	local LiberarLoop = RunService.RenderStepped:Connect(function()
		if not deathIsActive or Humanoid.Health <= 0 or not Humanoid.Parent then
			LiberarLoop:Disconnect(); return
		end
		RompeAgarresYLibera(Character)
	end)

	VidaCeroExtremo(Humanoid)

	local DiedConn = Humanoid.Died:Connect(function()
		DiedConn:Disconnect()
		LiberarLoop:Disconnect()
		btnDeath.Text = T("⏳ EN ESPERA...")
		LocalPlayer.CharacterAdded:Connect(function(NuevoPersonaje)
			NuevoPersonaje:WaitForChild("Humanoid", 3)
			task.wait(0.08)
			if deathIsActive then IniciarCicloDeath() end
		end)
	end)
end

local function ActivarDeathPor08s()
	if deathCicloActivo then return end
	deathCicloActivo = true
	deathIsActive = true
	btnDeath.Text = T("⚡ ACTIVO 0.8s")
	bordeActivo(btnDeath, true)
	IniciarCicloDeath()
	task.wait(0.8)
	deathIsActive = false
	deathCicloActivo = false
	btnDeath.Text = T("💀 DEATH 0.8s")
	bordeActivo(btnDeath, false)
end

btnDeath.MouseButton1Click:Connect(ActivarDeathPor08s)

-- ==================================================
-- 🔄 ESTADOS DE BOTONES
-- ==================================================
local function actualizarBotonToggle()
	if teleportEnabled then
		bordeActivo(btnToggle, true); btnToggle.Text = T("❌ DESACTIVAR")
	else
		bordeActivo(btnToggle, false); btnToggle.Text = T("✅ ACTIVAR")
	end
end

local function actualizarBotonHitbox()
	if hitboxActivo then
		bordeActivo(hitboxToggle, true)
		if seleccionMultipleActiva then
			hitboxToggle.Text = T("HITBOX ON x") .. contarSeleccionados()
		else
			hitboxToggle.Text = jugadorSeleccionado and ("HITBOX ON: " .. jugadorSeleccionado.Name:sub(1, 10)) or T("HITBOX ON: sin objetivo")
		end
	else
		bordeActivo(hitboxToggle, false)
		hitboxToggle.Text = T("HITBOX: OFF")
	end
end

local function actualizarBotonSeleccionMultiple()
	if seleccionMultipleActiva then
		bordeActivo(btnSeleccionMultiple, true)
		btnSeleccionMultiple.Text = T("☑️ MÚLTIPLE: ON (") .. contarSeleccionados() .. ")"
	else
		bordeActivo(btnSeleccionMultiple, false)
		btnSeleccionMultiple.Text = T("☑️ SELECCIÓN MÚLTIPLE: OFF")
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
			local baseTexto = plr.DisplayName .. " (@" .. plr.Name .. ")" .. (plr == LocalPlayer and T(" • TÚ") or "")
			if seleccionMultipleActiva and seleccionados[plr] then
				entry.BackgroundColor3 = COLOR_MULTI
				if nameLabel then nameLabel.Text = T("✓ ") .. baseTexto end
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
		statusLabel.Text = T("Objetivo: ") .. plr.Name
		statusLabel.TextColor3 = Color3.fromRGB(180, 255, 230)
	else
		statusLabel.Text = T("Status: Off | Objetivo: Ninguno")
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
		nameLabel.Text = plr.DisplayName .. " (@" .. plr.Name .. ")" .. (plr == LocalPlayer and T(" • TÚ") or "")
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
						if seleccionMultipleActiva then alternarSeleccionMultiple(plr) else establecerObjetivo(plr) end
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
		statusLabel.Text = T("🔄 Re-seleccionado: ") .. plr.Name
		statusLabel.TextColor3 = Color3.fromRGB(180, 255, 230)
	elseif reconexionPendienteMulti[plr.Name] then
		reconexionPendienteMulti[plr.Name] = nil
		if plr ~= LocalPlayer then
			seleccionados[plr] = true
			actualizarBotonSeleccionMultiple()
			actualizarBotonHitbox()
			refrescarColoresLista()
			statusLabel.Text = T("🔄 Re-añadido: ") .. plr.Name .. T(" (total ") .. contarSeleccionados() .. ")"
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
		statusLabel.Text = "⏳ " .. plr.Name .. T(" salió — esperando reconexión...")
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
			statusLabel.Text = (seleccionados[encontrado] and T("✓ Añadido: ") or T("➖ Quitado: ")) .. encontrado.Name .. T(" (total ") .. contarSeleccionados() .. ")"
			statusLabel.TextColor3 = Color3.fromRGB(180, 255, 230)
		else
			establecerObjetivo(encontrado)
			statusLabel.Text = T("Objetivo: ") .. encontrado.Name .. " (" .. encontrado.DisplayName .. ")"
			statusLabel.TextColor3 = Color3.fromRGB(180, 255, 230)
		end
	else
		statusLabel.Text = T("❌ No se encontró: ") .. txtManual.Text
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
			hitboxToggle.Text = seleccionMultipleActiva and T("SELECCIONA ALGUIEN") or T("AVATAR NO ENCONTRADO")
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
		if jugadorSeleccionado and jugadorSeleccionado ~= LocalPlayer then seleccionados[jugadorSeleccionado] = true end
		bordeActivo(btnSeleccionMultiple, true)
		statusLabel.Text = T("☑️ Modo múltiple ON — toca jugadores en la lista")
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
		statusLabel.Text = T("Status: Listo")
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
		statusLabel.Text = T("❌ Sin personaje"); task.delay(1.5, function() statusLabel.Text = T("Status: Listo") end); return
	end
	lastPosition = character.HumanoidRootPart.Position
	statusLabel.Text = T("✅ Posición guardada!")
	task.wait(0.3)
	character:PivotTo(CFrame.new(lastPosition + Vector3.new(0, DISTANCIA_TP_GUARDADO, 0)))
	statusLabel.Text = T("✅ ¡TP -500!")
	task.delay(1.2, function() statusLabel.Text = T("Status: Listo") end)
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
			statusLabel.Text = T("❌ Nadie cerca")
			task.delay(1.2, function() statusLabel.Text = T("Status: Listo") end)
			return
		end
		autoMode = true
		targetSaved = target
		statusLabel.Text = T("Fijado: ") .. target.Name
		btnAbrazo.Text = T("🛑 DETENER")
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
			btnAbrazo.Text = T("🎯 ABRAZO BUG")
			bordeActivo(btnAbrazo, false)
			statusLabel.Text = T("Status: Listo")
		end)
	else
		autoMode = false
		targetSaved = nil
		btnAbrazo.Text = T("🎯 ABRAZO BUG")
		bordeActivo(btnAbrazo, false)
		statusLabel.Text = T("Status: Listo")
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
		btnToggleLista.Text = T("📋 Cerrar")
		actualizarListaJugadores()
	else
		areaLista.Size = UDim2.new(1, 0, 0, 0)
		framePrincipal.Size = UDim2.new(0, ancho, 0, altoBase + ALTO_TITULO)
		btnToggleLista.Text = T("📋 Lista")
	end
end)

btnSeleccionarCercano.MouseButton1Click:Connect(function()
	local cercano = obtenerJugadorMasCercano()
	if cercano then
		if seleccionMultipleActiva then
			alternarSeleccionMultiple(cercano)
			statusLabel.Text = (seleccionados[cercano] and T("✓ Añadido cercano: ") or T("➖ Quitado: ")) .. cercano.Name .. T(" (total ") .. contarSeleccionados() .. ")"
			statusLabel.TextColor3 = Color3.fromRGB(180, 255, 230)
		else
			establecerObjetivo(cercano)
		end
	else
		establecerObjetivo(nil)
		statusLabel.Text = T("No se encontró jugador")
		statusLabel.TextColor3 = Color3.fromRGB(255, 120, 120)
	end
end)

btnToggle.MouseButton1Click:Connect(function()
	if teleportEnabled then
		teleportEnabled = false
		statusLabel.Text = T("Status: Off | Objetivo: ") .. (jugadorSeleccionado and jugadorSeleccionado.Name or T("Ninguno"))
		statusLabel.TextColor3 = Color3.fromRGB(255, 120, 120)
		restaurarPosicionOriginal()
	else
		if not hayObjetivosActivos() then
			statusLabel.Text = T("¡Selecciona un jugador primero!")
			statusLabel.TextColor3 = Color3.fromRGB(255, 120, 120)
			return
		end
		teleportEnabled = true
		if seleccionMultipleActiva then
			statusLabel.Text = T("✅ TP visual ON x") .. contarSeleccionados()
		else
			statusLabel.Text = T("✅ Activo | Objetivo: ") .. jugadorSeleccionado.Name
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

-- ==================================================
-- 🌐 APLICAR IDIOMA A TODA LA GUI (con memoria del original para el botón 🌐)
-- ==================================================
local function aplicarIdiomaATodo()
	for _, obj in ipairs(screenGui:GetDescendants()) do
		if obj:IsA("TextLabel") or obj:IsA("TextButton") or obj:IsA("TextBox") then
			local orig = obj:GetAttribute("_orig")
			if orig then obj.Text = T(orig) end
			if obj:IsA("TextBox") then
				local origp = obj:GetAttribute("_origp")
				if origp then obj.PlaceholderText = T(origp) end
			end
		end
	end
end

-- Guardamos el texto original (en español) y aplicamos el idioma detectado
for _, obj in ipairs(screenGui:GetDescendants()) do
	if obj:IsA("TextLabel") or obj:IsA("TextButton") or obj:IsA("TextBox") then
		obj:SetAttribute("_orig", obj.Text)
		obj.Text = T(obj.Text)
		if obj:IsA("TextBox") then
			obj:SetAttribute("_origp", obj.PlaceholderText)
			obj.PlaceholderText = T(obj.PlaceholderText)
		end
	end
end

-- Refresca los textos dinámicos (botones con estado) tras cambiar de idioma
local function refrescarTextosDinamicos()
	actualizarBotonToggle()
	actualizarBotonHitbox()
	actualizarBotonSeleccionMultiple()
	if deathCicloActivo then btnDeath.Text = T("⚡ ACTIVO 0.8s") else btnDeath.Text = T("💀 DEATH 0.8s") end
	if autoMode then btnAbrazo.Text = T("🛑 DETENER") else btnAbrazo.Text = T("🎯 ABRAZO BUG") end
	if scooterActivo then btnScooter.Text = T("🛴 SCOOTER: ON") else btnScooter.Text = T("🛴 SCOOTER: OFF") end
	if listaAbierta then btnToggleLista.Text = T("📋 Cerrar") else btnToggleLista.Text = T("📋 Lista") end
	actualizarListaJugadores()
end

-- 🌐 Botón de idioma: cicla ES → EN → PT → JA → ZH y re-traduce todo al instante
local ORDEN_IDIOMAS = { "es", "en", "pt", "ja", "zh" }
btnIdioma.MouseButton1Click:Connect(function()
	local idx = 1
	for i, cod in ipairs(ORDEN_IDIOMAS) do
		if cod == IDIOMA then idx = i; break end
	end
	idx = idx % #ORDEN_IDIOMAS + 1
	IDIOMA = ORDEN_IDIOMAS[idx]
	aplicarIdiomaATodo()
	refrescarTextosDinamicos()
	statusLabel.Text = "🌐 " .. IDIOMA:upper()
	statusLabel.TextColor3 = Color3.fromRGB(180, 255, 230)
	print("🌐 Idioma cambiado a:", IDIOMA)
end)

print("✅ AA.toki v36 (botones pequeños + traductor FIX, idioma=" .. IDIOMA .. ") cargado correctamente")
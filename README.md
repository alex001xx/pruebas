local WindUI = loadstring(game:HttpGet("https://github.com/Footagesus/WindUI/releases/latest/download/main.lua"))()
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local TeleportService = game:GetService("TeleportService")
local StarterGui = game:GetService("StarterGui")
local UserInputService = game:GetService("UserInputService")
local LocalPlayer = Players.LocalPlayer
local PlayerName, DisplayName, UserId = LocalPlayer.Name, LocalPlayer.DisplayName, LocalPlayer.UserId
local AntiAFKEnabled, NoFrictionEnabled, NoPushEnabled, NoKnockbackEnabled = false, false, false, false
local FlyEnabled, FlySpeed = false, 60
local FlyConn, FlyVel, FlyGyro, NoclipEnabled, NoclipConn = nil, nil, nil, false, nil
local ESPEnabled, ESPConn, FullbrightEnabled, AutoRejoinEnabled = false, nil, false, false
local AntiVoidEnabled, AntiRagdollEnabled, InfJumpEnabled, InfJumpConn = false, false, false, nil
local FpsBoostEnabled = false
local LastSafePos = Vector3.new(0,10,0)
local SpawnCFrame = CFrame.new()
local SpectateName, TPPlayerName, SpectateDropdown, TPPlayerDropdown, CoordsParagraph = nil, nil, nil, nil, nil
local GodModeEnabled, ClickTPEnabled, AutoJumpEnabled, WalkOnWaterEnabled = false, false, false, false
local FollowPlayerEnabled, SpinBotEnabled, FreezePositionEnabled, InstantRespawnEnabled = false, false, false, false
local FallSpeedCap, SavedPosition, CoordsText, ClickTPConn, SpinConn = 200, nil, "", nil, nil
local AutoWalkEnabled, AutoWalkConn = false, nil
local TPWalkEnabled, TPWalkSpeed, TPWalkConn = false, 0.30, nil
local ChatSpamEnabled, DanceSpamEnabled, RainbowEnabled, DrunkCamEnabled = false, false, false, false
local PlayDeadEnabled, ChatEchoEnabled, BigHeadEnabled, ScreenShakeEnabled = false, false, false, false
local SpamText, MorphName, MorphDropdown, RainbowConn = "Hola! :D", nil, nil, nil
local EchoConns, EchoPlayerAddedConn, ChatSpamKill, DanceKill, OriginalHeadSize = {}, nil, false, false, nil
local InvisibleEnabled = false
local AutoClickerEnabled, AutoClickerCPS = false, 10
local NoFallDamageEnabled, NoFallConn = false, nil
local AntiAFKConn = nil
local TargetName = nil
local TargetDropdown, GameTargetDropdown, TargetInfoParagraph = nil, nil, nil
local Lighting = game:GetService("Lighting")
local OriginalLight = {ClockTime=Lighting.ClockTime, Brightness=Lighting.Brightness, Ambient=Lighting.Ambient, OutdoorAmbient=Lighting.OutdoorAmbient, FogEnd=Lighting.FogEnd, GlobalShadows=Lighting.GlobalShadows, ExposureCompensation=Lighting.ExposureCompensation}
local GameSettings = UserSettings().GameSettings
local OriginalQuality = Enum.SavedQualitySetting.Automatic
pcall(function() OriginalQuality = GameSettings.SavedQualityLevel end)

local FlashAttackEnabled = false
local FlashMultiplier = 5
local FlashConn = nil

local SitProtectorEnabled = false
local SitConexiones = {}
local SitEstadosOriginales = {}
local SitGui = nil

local Character, Humanoid, RootPart
local function UpdateChar()
    Character = LocalPlayer.Character or LocalPlayer.CharacterAdded:Wait()
    Humanoid = Character:WaitForChild("Humanoid")
    RootPart = Character:WaitForChild("HumanoidRootPart")
    SpawnCFrame = RootPart.CFrame
end
UpdateChar()
LocalPlayer.CharacterAdded:Connect(UpdateChar)

local function GetPlayerNames(includeNone)
    local names = {}
    if includeNone ~= false then table.insert(names, "Nadie (detener)") end
    for _, p in ipairs(Players:GetPlayers()) do if p ~= LocalPlayer then table.insert(names, p.Name) end end
    if #names == 0 then table.insert(names, "No hay jugadores") end
    return names
end
local function SayChat(text) pcall(function() game:GetService("ReplicatedStorage").DefaultChatSystemChatEvents.SayMessageRequest:FireServer(text, "All") end) end
local function TeleportToCoords(text)
    if not text or text == "" then WindUI:Notify({Title="Error", Content="Texto vacío", Duration=2}) return end
    local x,y,z = text:match("([%-%d%.]+)[,%s]+([%-%d%.]+)[,%s]+([%-%d%.]+)")
    x,y,z = tonumber(x), tonumber(y), tonumber(z)
    if x and y and z and RootPart then
        RootPart.CFrame = CFrame.new(x,y,z); RootPart.Velocity = Vector3.new(0,0,0)
        WindUI:Notify({Title="TP", Content="X:"..x.." Y:"..y.." Z:"..z, Duration=2})
    else WindUI:Notify({Title="Error", Content="Formato: X, Y, Z", Duration=3}) end
end

local function EnsureFlyMovers()
    if not RootPart then return false end
    if not RootPart:FindFirstChild("WindFlyVel") then
        local bv = Instance.new("BodyVelocity"); bv.Name="WindFlyVel"; bv.MaxForce=Vector3.new(9e9,9e9,9e9); bv.Velocity=Vector3.new(0,0,0); bv.Parent=RootPart; FlyVel=bv
    else FlyVel = RootPart.WindFlyVel end
    if not RootPart:FindFirstChild("WindFlyGyro") then
        local bg = Instance.new("BodyGyro"); bg.Name="WindFlyGyro"; bg.P=9e4; bg.MaxTorque=Vector3.new(9e9,9e9,9e9); bg.Parent=RootPart; FlyGyro=bg
    else FlyGyro = RootPart.WindFlyGyro end
    if Humanoid then Humanoid.PlatformStand=true; Humanoid.GravityScale=0 end
    return true
end
local function StartFly()
    FlyEnabled=true
    if not FlyConn then
        FlyConn = RunService.RenderStepped:Connect(function()
            if not FlyEnabled or not EnsureFlyMovers() then return end
            local cf = workspace.CurrentCamera.CFrame
            local move = Vector3.new(0,0,0)
            if UserInputService:IsKeyDown(Enum.KeyCode.W) then move=move+cf.LookVector end
            if UserInputService:IsKeyDown(Enum.KeyCode.S) then move=move-cf.LookVector end
            if UserInputService:IsKeyDown(Enum.KeyCode.A) then move=move-cf.RightVector end
            if UserInputService:IsKeyDown(Enum.KeyCode.D) then move=move+cf.RightVector end
            if UserInputService:IsKeyDown(Enum.KeyCode.Space) then move=move+Vector3.new(0,1,0) end
            if UserInputService:IsKeyDown(Enum.KeyCode.LeftControl) then move=move-Vector3.new(0,1,0) end
            if move.Magnitude>0 then move=move.Unit end
            FlyVel.Velocity = move*FlySpeed; FlyGyro.CFrame=cf
        end)
    end
end
local function StopFly()
    FlyEnabled=false
    if FlyConn then FlyConn:Disconnect(); FlyConn=nil end
    if RootPart then
        local bv=RootPart:FindFirstChild("WindFlyVel"); if bv then bv:Destroy() end
        local bg=RootPart:FindFirstChild("WindFlyGyro"); if bg then bg:Destroy() end
    end
    if Humanoid then Humanoid.PlatformStand=false; Humanoid.GravityScale=1 end
end
local function SetNoclip(state)
    NoclipEnabled=state
    if state then
        if not NoclipConn then NoclipConn=RunService.Stepped:Connect(function()
            if not NoclipEnabled or not Character then return end
            for _,v in ipairs(Character:GetDescendants()) do if v:IsA("BasePart") then v.CanCollide=false end end
        end) end
    else if NoclipConn then NoclipConn:Disconnect(); NoclipConn=nil end end
end
local function SetInfJump(state)
    InfJumpEnabled=state
    if state then
        if not InfJumpConn then InfJumpConn=UserInputService.JumpRequest:Connect(function()
            if Humanoid then Humanoid:ChangeState(Enum.HumanoidStateType.Jumping) end
        end) end
    else if InfJumpConn then InfJumpConn:Disconnect(); InfJumpConn=nil end end
end
local function SetTPWalk(s)
    TPWalkEnabled = s
    if s then
        if not TPWalkConn then
            TPWalkConn = RunService.Heartbeat:Connect(function()
                if not TPWalkEnabled then return end
                local char = LocalPlayer.Character
                if not char then return end
                local hum = char:FindFirstChildOfClass("Humanoid")
                local hrp = char:FindFirstChild("HumanoidRootPart")
                if hum and hrp and hum.MoveDirection.Magnitude > 0.1 then
                    hrp.CFrame = hrp.CFrame + (hum.MoveDirection * TPWalkSpeed)
                end
            end)
        end
    else
        if TPWalkConn then TPWalkConn:Disconnect(); TPWalkConn = nil end
    end
end

local function SetFlashAttack(s)
    FlashAttackEnabled = s
    if s then
        if not FlashConn then
            FlashConn = RunService.Heartbeat:Connect(function()
                if not FlashAttackEnabled or not Character then return end
                local tool = Character:FindFirstChildWhichIsA("Tool")
                if not tool then return end
                pcall(function()
                    for _ = 1, FlashMultiplier do tool:Activate() end
                    for _, v in pairs(tool:GetDescendants()) do
                        if v:IsA("NumberValue") or v:IsA("IntValue") then
                            local name = v.Name:lower()
                            if name:find("cooldown") or name:find("delay") or name:find("rate") or name:find("time") then v.Value = 0 end
                        end
                    end
                end)
            end)
        end
    else
        if FlashConn then FlashConn:Disconnect(); FlashConn = nil end
    end
end

local function GuardarEstadosOriginalesSit(Personaje)
    if not Personaje then return end
    local Hum = Personaje:FindFirstChildWhichIsA("Humanoid")
    if not Hum then return end
    SitEstadosOriginales = {
        Sit = false, PlatformStand = false, AutoRotate = true,
        Seated = Hum:GetStateEnabled(Enum.HumanoidStateType.Seated),
        FallingDown = Hum:GetStateEnabled(Enum.HumanoidStateType.FallingDown),
        Ragdoll = Hum:GetStateEnabled(Enum.HumanoidStateType.Ragdoll),
        PlatformStanding = Hum:GetStateEnabled(Enum.HumanoidStateType.PlatformStanding),
    }
end
local function RestaurarTodoNormalSit()
    local Pj = LocalPlayer.Character
    if not Pj then return end
    local Hum = Pj:FindFirstChildWhichIsA("Humanoid")
    if not Hum then return end
    Hum.Sit = SitEstadosOriginales.Sit
    Hum.PlatformStand = SitEstadosOriginales.PlatformStand
    Hum.AutoRotate = SitEstadosOriginales.AutoRotate
    Hum:SetStateEnabled(Enum.HumanoidStateType.Seated, SitEstadosOriginales.Seated)
    Hum:SetStateEnabled(Enum.HumanoidStateType.FallingDown, SitEstadosOriginales.FallingDown)
    Hum:SetStateEnabled(Enum.HumanoidStateType.Ragdoll, SitEstadosOriginales.Ragdoll)
    Hum:SetStateEnabled(Enum.HumanoidStateType.PlatformStanding, SitEstadosOriginales.PlatformStanding)
end
local function CrearSitGui()
    if SitGui then return end
    local Gui = Instance.new("ScreenGui")
    Gui.Name = "SitProtector"; Gui.Parent = LocalPlayer:WaitForChild("PlayerGui")
    Gui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling; Gui.ResetOnSpawn = false
    local Marco = Instance.new("Frame")
    Marco.Size = UDim2.new(0, 120, 0, 55); Marco.Position = UDim2.new(0.02, 0, 0.02, 0)
    Marco.BackgroundColor3 = Color3.fromRGB(250, 235, 170); Marco.BorderColor3 = Color3.fromRGB(245, 210, 90)
    Marco.BorderSizePixel = 2; Marco.Active = true; Marco.Draggable = true; Marco.Parent = Gui
    local Boton = Instance.new("TextButton")
    Boton.Size = UDim2.new(0, 90, 0, 35); Boton.Position = UDim2.new(0.5, -45, 0.5, -17)
    Boton.BackgroundColor3 = Color3.fromRGB(245, 210, 90); Boton.TextColor3 = Color3.fromRGB(70, 60, 40)
    Boton.Font = Enum.Font.GothamBold; Boton.TextSize = 12; Boton.Text = "OFF"
    Boton.AutoLocalize = false; Boton.Parent = Marco
    SitGui = {Gui=Gui, Marco=Marco, Boton=Boton}
end
local function SetSitProtector(s)
    if s and SitProtectorEnabled then return end
    SitProtectorEnabled = s
    CrearSitGui()
    if s then
        SitGui.Boton.Text = "ON"
        SitGui.Boton.BackgroundColor3 = Color3.fromRGB(170, 245, 180)
        GuardarEstadosOriginalesSit(LocalPlayer.Character)
        SitConexiones.Bucle = RunService.Heartbeat:Connect(function()
            local Pj = LocalPlayer.Character
            if not Pj then return end
            local Hum = Pj:FindFirstChildWhichIsA("Humanoid")
            local Raiz = Pj:FindFirstChild("HumanoidRootPart")
            if not Hum or not Raiz then return end
            Hum.Sit = true
            Hum:SetStateEnabled(Enum.HumanoidStateType.Seated, false)
            Hum:SetStateEnabled(Enum.HumanoidStateType.FallingDown, false)
            Hum:SetStateEnabled(Enum.HumanoidStateType.Ragdoll, false)
            Hum:SetStateEnabled(Enum.HumanoidStateType.PlatformStanding, false)
            Hum.AutoRotate = true; Hum.PlatformStand = false
            if math.abs(Raiz.RotVelocity.Y) > 8 then
                Raiz.RotVelocity = Vector3.new(0, math.sign(Raiz.RotVelocity.Y) * 8, 0)
            end
            for _, Pieza in Pj:GetChildren() do
                if Pieza:IsA("Weld") or Pieza:IsA("WeldConstraint") or Pieza:IsA("Motor6D") then
                    if Pieza.Name ~= "RootJoint" and Pieza.Name ~= "Neck" and Pieza.Name ~= "Waist"
                    and not Pieza:FindFirstAncestorWhichIsA("Tool") then Pieza:Destroy() end
                end
            end
        end)
    else
        if SitGui then
            SitGui.Boton.Text = "OFF"
            SitGui.Boton.BackgroundColor3 = Color3.fromRGB(245, 210, 90)
        end
        if SitConexiones.Bucle then SitConexiones.Bucle:Disconnect(); SitConexiones.Bucle = nil end
        RestaurarTodoNormalSit()
    end
end
LocalPlayer.CharacterAdded:Connect(function()
    task.wait(0.2)
    if SitGui and SitProtectorEnabled then SitGui.Gui.Parent = LocalPlayer:WaitForChild("PlayerGui") end
end)

local function SetESP(state)
    ESPEnabled=state
    if state then
        if not ESPConn then
            ESPConn=RunService.Heartbeat:Connect(function()
                if not ESPEnabled then return end
                for _,plr in ipairs(Players:GetPlayers()) do
                    if plr~=LocalPlayer and plr.Character then
                        local char=plr.Character
                        if not char:FindFirstChild("WindESP_HL") then
                            local hl=Instance.new("Highlight"); hl.Name="WindESP_HL"; hl.FillColor=Color3.fromRGB(255,50,50); hl.OutlineColor=Color3.fromRGB(255,255,255); hl.FillTransparency=0.6; hl.DepthMode=Enum.HighlightDepthMode.AlwaysOnTop; hl.Parent=char
                        end
                        local head=char:FindFirstChild("Head")
                        if head then
                            local bg=head:FindFirstChild("WindESP_BG")
                            if not bg then
                                bg=Instance.new("BillboardGui"); bg.Name="WindESP_BG"; bg.Size=UDim2.new(0,220,0,40); bg.StudsOffset=Vector3.new(0,2.5,0); bg.AlwaysOnTop=true
                                local tl=Instance.new("TextLabel"); tl.Name="TextLabel"; tl.Size=UDim2.new(1,0,1,0); tl.BackgroundTransparency=1; tl.TextColor3=Color3.fromRGB(255,255,255); tl.TextStrokeTransparency=0; tl.Font=Enum.Font.SourceSansBold; tl.TextScaled=true; tl.Parent=bg; bg.Parent=head
                            end
                            local dist = RootPart and math.floor((RootPart.Position-head.Position).Magnitude) or 0
                            bg.TextLabel.Text = plr.Name.."  ["..dist.." m]"
                        end
                    end
                end
            end)
        end
    else
        if ESPConn then ESPConn:Disconnect(); ESPConn=nil end
        for _,plr in ipairs(Players:GetPlayers()) do
            if plr.Character then
                local hl=plr.Character:FindFirstChild("WindESP_HL"); if hl then hl:Destroy() end
                local head=plr.Character:FindFirstChild("Head")
                if head then local bg=head:FindFirstChild("WindESP_BG"); if bg then bg:Destroy() end end
            end
        end
    end
end
local function SetFullbright(state)
    FullbrightEnabled=state
    if state then
        Lighting.ClockTime=14; Lighting.Brightness=2; Lighting.Ambient=Color3.fromRGB(255,255,255); Lighting.OutdoorAmbient=Color3.fromRGB(255,255,255); Lighting.FogEnd=100000; Lighting.GlobalShadows=false; Lighting.ExposureCompensation=0.5
        for _,e in ipairs(Lighting:GetChildren()) do pcall(function() e.Enabled=false end) end
    else
        Lighting.ClockTime=OriginalLight.ClockTime; Lighting.Brightness=OriginalLight.Brightness; Lighting.Ambient=OriginalLight.Ambient; Lighting.OutdoorAmbient=OriginalLight.OutdoorAmbient; Lighting.FogEnd=OriginalLight.FogEnd; Lighting.GlobalShadows=OriginalLight.GlobalShadows; Lighting.ExposureCompensation=OriginalLight.ExposureCompensation
        for _,e in ipairs(Lighting:GetChildren()) do pcall(function() e.Enabled=true end) end
    end
end
local function SetFpsBoost(state)
    FpsBoostEnabled=state
    if state then
        pcall(function() GameSettings.SavedQualityLevel=Enum.SavedQualitySetting.QualityLevel1 end)
        pcall(function() local t=workspace:FindFirstChildOfClass("Terrain"); if t then t.WaterWaveSize=0; t.WaterWaveSpeed=0 end end)
    else pcall(function() GameSettings.SavedQualityLevel=OriginalQuality end) end
end
local function ServerHop()
    task.spawn(function()
        local ok=pcall(function()
            local Http=game:GetService("HttpService")
            local response=Http:JSONDecode(Http:GetAsync("https://games.roproxy.com/v1/games/"..game.PlaceId.."/servers/Public?sortOrder=Asc&limit=100"))
            local candidates={}
            if response and response.data then for _,s in ipairs(response.data) do if s.id~=game.JobId and s.playing and s.maxPlayers and s.playing<s.maxPlayers then table.insert(candidates,s) end end end
            if #candidates>0 then
                local chosen=candidates[math.random(1,#candidates)]
                WindUI:Notify({Title="Server Hop", Content="Conectando...", Duration=3})
                TeleportService:TeleportToPlaceInstance(game.PlaceId, chosen.id)
            else WindUI:Notify({Title="Server Hop", Content="No hay servidores", Duration=3}) end
        end)
        if not ok then WindUI:Notify({Title="Server Hop", Content="Error (proxy caído?)", Duration=3}) end
    end)
end
local function SetClickTP(s)
    ClickTPEnabled=s
    if s then
        if not ClickTPConn then ClickTPConn=UserInputService.InputBegan:Connect(function(input,gp)
            if not ClickTPEnabled or gp then return end
            if input.UserInputType==Enum.UserInputType.MouseButton2 then
                local Mouse=LocalPlayer:GetMouse()
                if RootPart and Mouse and Mouse.Hit then RootPart.CFrame=CFrame.new(Mouse.Hit.Position+Vector3.new(0,3,0)) end
            end
        end) end
    else if ClickTPConn then ClickTPConn:Disconnect(); ClickTPConn=nil end end
end
local function SetSpinBot(s)
    SpinBotEnabled=s
    if s then
        if not SpinConn then SpinConn=RunService.RenderStepped:Connect(function()
            if not SpinBotEnabled or not RootPart then return end
            RootPart.CFrame=RootPart.CFrame*CFrame.Angles(0,math.rad(15),0)
        end) end
    else if SpinConn then SpinConn:Disconnect(); SpinConn=nil end end
end
local function SetFreeze(s) FreezePositionEnabled=s; if RootPart then pcall(function() RootPart.Anchored=s end) end end
local function SetAutoWalk(s)
    AutoWalkEnabled=s
    if s then
        if not AutoWalkConn then AutoWalkConn=RunService.RenderStepped:Connect(function()
            if not AutoWalkEnabled or not Humanoid then return end
            local cam=workspace.CurrentCamera
            if cam then local dir=cam.CFrame.LookVector*Vector3.new(1,0,1); if dir.Magnitude>0 then Humanoid:Move(dir.Unit,false) end end
        end) end
    else
        if AutoWalkConn then AutoWalkConn:Disconnect(); AutoWalkConn=nil end
        if Humanoid then Humanoid:Move(Vector3.new(0,0,0)) end
    end
end
local function RemoveParticles()
    local count=0
    pcall(function() for _,v in ipairs(workspace:GetDescendants()) do if v:IsA("ParticleEmitter") or v:IsA("Trail") or v:IsA("Smoke") or v:IsA("Fire") or v:IsA("Sparkles") or v:IsA("Beam") or v:IsA("Explosion") then v:Destroy(); count=count+1 end end end)
    WindUI:Notify({Title="Partículas", Content="Eliminados "..count.." efectos", Duration=2})
end
local function SetChatSpam(s)
    if s and ChatSpamEnabled then return end
    ChatSpamEnabled=s
    if s then ChatSpamKill=false; task.spawn(function() while ChatSpamEnabled and not ChatSpamKill do SayChat(SpamText); task.wait(1.2) end end)
    else ChatSpamKill=true end
end
local function SetDanceSpam(s)
    if s and DanceSpamEnabled then return end
    DanceSpamEnabled=s
    if s then DanceKill=false; local d={"/e dance","/e dance2","/e dance3","/e wave","/e laugh"}; task.spawn(function() while DanceSpamEnabled and not DanceKill do SayChat(d[math.random(1,#d)]); task.wait(1.5) end end)
    else DanceKill=true end
end
local function FakeKick()
    task.spawn(function() pcall(function()
        local sg=Instance.new("ScreenGui"); sg.Name="WindFakeKick"; sg.ResetOnSpawn=false; sg.DisplayOrder=9999
        local fr=Instance.new("Frame"); fr.Size=UDim2.fromScale(1,1); fr.BackgroundColor3=Color3.fromRGB(120,0,0); fr.BorderSizePixel=0; fr.Parent=sg
        local tl=Instance.new("TextLabel"); tl.Size=UDim2.fromScale(0.8,0.4); tl.Position=UDim2.fromScale(0.1,0.3); tl.BackgroundTransparency=1; tl.Text="KICKED\n\nYou were kicked from this experience.\nReason: Cheating / Exploiting"; tl.TextColor3=Color3.fromRGB(255,255,255); tl.TextScaled=true; tl.Font=Enum.Font.SourceSansBold; tl.Parent=fr
        local ok,core=pcall(function() return game:GetService("CoreGui") end)
        sg.Parent=(ok and core) or LocalPlayer:WaitForChild("PlayerGui")
        task.wait(4); sg:Destroy()
    end) end)
end
local function SetRainbow(s)
    RainbowEnabled=s
    if s then
        if not RainbowConn then
            local hue=0
            RainbowConn=RunService.Heartbeat:Connect(function()
                if not RainbowEnabled then return end
                hue=(hue+0.015)%1; local c=Color3.fromHSV(hue,1,1)
                if Character then
                    local bc=Character:FindFirstChildOfClass("BodyColors")
                    if bc then pcall(function() bc.HeadColor=c; bc.TorsoColor=c; bc.LeftArmColor=c; bc.RightArmColor=c; bc.LeftLegColor=c; bc.RightLegColor=c end)
                    else for _,v in ipairs(Character:GetDescendants()) do if v:IsA("BasePart") and v.Name~="HumanoidRootPart" then pcall(function() v.Color=c end) end end end
                end
            end)
        end
    else if RainbowConn then RainbowConn:Disconnect(); RainbowConn=nil end end
end
local function SetDrunkCam(s)
    if s and DrunkCamEnabled then return end
    DrunkCamEnabled=s
    if s then pcall(function() RunService:BindToRenderStep("WindDrunkCam", Enum.RenderPriority.Camera.Value+1, function()
        if not DrunkCamEnabled then return end
        local cam=workspace.CurrentCamera
        if cam then cam.CFrame=cam.CFrame*CFrame.Angles(math.sin(tick()*3)*0.06, math.cos(tick()*2.5)*0.06, math.sin(tick()*4)*0.04) end
    end) end)
    else pcall(function() RunService:UnbindFromRenderStep("WindDrunkCam") end) end
end
local function SetPlayDead(s)
    PlayDeadEnabled=s
    if Humanoid then pcall(function()
        if s then Humanoid.PlatformStand=true; Humanoid:ChangeState(Enum.HumanoidStateType.Physics)
        else Humanoid.PlatformStand=false; Humanoid:ChangeState(Enum.HumanoidStateType.GettingUp) end
    end) end
end
local function HookEchoPlayer(plr)
    local conn=plr.Chatted:Connect(function(msg) if not ChatEchoEnabled or plr==LocalPlayer then return end; task.wait(0.3); SayChat(msg) end)
    table.insert(EchoConns, conn)
end
local function SetChatEcho(s)
    if s and ChatEchoEnabled then return end
    ChatEchoEnabled=s
    if s then
        for _,plr in ipairs(Players:GetPlayers()) do if plr~=LocalPlayer then HookEchoPlayer(plr) end end
        if not EchoPlayerAddedConn then EchoPlayerAddedConn=Players.PlayerAdded:Connect(function(plr) if ChatEchoEnabled then HookEchoPlayer(plr) end end) end
    else
        for _,c in ipairs(EchoConns) do pcall(function() c:Disconnect() end) end; EchoConns={}
        if EchoPlayerAddedConn then EchoPlayerAddedConn:Disconnect(); EchoPlayerAddedConn=nil end
    end
end
local function SetBigHead(s)
    BigHeadEnabled=s
    pcall(function()
        if Humanoid and Humanoid.HeadScale then Humanoid.HeadScale=s and 3 or 1 end
        local head=Character and Character:FindFirstChild("Head")
        if head and head:IsA("BasePart") then if not OriginalHeadSize then OriginalHeadSize=head.Size end; head.Size=s and (OriginalHeadSize*3) or OriginalHeadSize end
    end)
end
local function SetScreenShake(s)
    if s and ScreenShakeEnabled then return end
    ScreenShakeEnabled=s
    if s then pcall(function() RunService:BindToRenderStep("WindShake", Enum.RenderPriority.Camera.Value+2, function()
        if not ScreenShakeEnabled then return end
        local cam=workspace.CurrentCamera
        if cam then cam.CFrame=cam.CFrame*CFrame.Angles((math.random()-0.5)*0.09,(math.random()-0.5)*0.09,(math.random()-0.5)*0.05) end
    end) end)
    else pcall(function() RunService:UnbindFromRenderStep("WindShake") end) end
end
local function MorphAsPlayer()
    task.spawn(function()
        if not MorphName or MorphName=="No hay jugadores" then WindUI:Notify({Title="Morph", Content="Selecciona un jugador", Duration=2}); return end
        local target=Players:FindFirstChild(MorphName)
        if target and Humanoid then
            local ok=pcall(function() local desc=Players:GetHumanoidDescriptionFromUserId(target.UserId); Humanoid:ApplyDescription(desc) end)
            WindUI:Notify({Title=ok and "Morph" or "Error", Content=ok and "Ahora te ves como "..MorphName or "No se pudo aplicar", Duration=2})
        else WindUI:Notify({Title="Error", Content="Jugador no disponible", Duration=2}) end
    end)
end

local function SetAutoClicker(s)
    AutoClickerEnabled = s
    if s then
        task.spawn(function()
            while AutoClickerEnabled do
                pcall(function()
                    local VIM = game:GetService("VirtualInputManager")
                    local ml = UserInputService:GetMouseLocation()
                    VIM:SendMouseButtonEvent(ml.X, ml.Y, 0, true, game, 0)
                    task.wait(0.02)
                    VIM:SendMouseButtonEvent(ml.X, ml.Y, 0, false, game, 0)
                end)
                task.wait(math.max(0.05, 1 / AutoClickerCPS))
            end
        end)
    end
end

local function SetNoFallDamage(s)
    NoFallDamageEnabled = s
    if s then
        if not NoFallConn and Humanoid then
            NoFallConn = Humanoid.StateChanged:Connect(function(_, new)
                if NoFallDamageEnabled and new == Enum.HumanoidStateType.Landed then
                    pcall(function() if Humanoid then Humanoid.Health = Humanoid.MaxHealth end end)
                end
            end)
        end
    else
        if NoFallConn then NoFallConn:Disconnect(); NoFallConn = nil end
    end
end
LocalPlayer.CharacterAdded:Connect(function()
    task.wait(0.5)
    if NoFallDamageEnabled then
        if NoFallConn then NoFallConn:Disconnect(); NoFallConn = nil end
        SetNoFallDamage(true)
    end
end)

local function SetAntiAFK(s)
    AntiAFKEnabled = s
    if s then
        if not AntiAFKConn then
            AntiAFKConn = LocalPlayer.Idled:Connect(function()
                pcall(function()
                    local vu = game:GetService("VirtualUser")
                    vu:CaptureController()
                    vu:ClickButton2(Vector2.new())
                end)
            end)
        end
    else
        if AntiAFKConn then AntiAFKConn:Disconnect(); AntiAFKConn = nil end
    end
end

local function TPAllToMe()
    if not RootPart then return end
    local count = 0
    for _, p in ipairs(Players:GetPlayers()) do
        if p ~= LocalPlayer and p.Character and p.Character:FindFirstChild("HumanoidRootPart") then
            pcall(function() p.Character.HumanoidRootPart.CFrame = RootPart.CFrame * CFrame.new(math.random(-6,6), 0, math.random(-6,6)) end)
            count = count + 1
        end
    end
    WindUI:Notify({Title="TP All", Content="Teleportados "..count.." jugadores a ti", Duration=3})
end

-- 🎯 FUNCIONES DE TARGET (objetivo/jugador)
local function ValidTarget()
    if not TargetName or TargetName=="" or TargetName=="No hay jugadores" then return nil end
    return Players:FindFirstChild(TargetName)
end
local function FocusTarget()
    local t = ValidTarget()
    if not t then WindUI:Notify({Title="Target", Content="Selecciona un jugador", Duration=2}); return end
    if t.Character and t.Character:FindFirstChild("Humanoid") then
        pcall(function() workspace.CurrentCamera.CameraSubject = t.Character.Humanoid end)
        WindUI:Notify({Title="Focus", Content="Cámara en "..TargetName, Duration=2})
    else WindUI:Notify({Title="Error", Content="Target no disponible", Duration=2}) end
end
local function StopFocus()
    pcall(function() workspace.CurrentCamera.CameraSubject = Humanoid end)
    WindUI:Notify({Title="Focus", Content="Detenido", Duration=2})
end
local function TPToTarget()
    local t = ValidTarget()
    if not t then WindUI:Notify({Title="TP", Content="Selecciona un jugador", Duration=2}); return end
    if t.Character and t.Character:FindFirstChild("HumanoidRootPart") and RootPart then
        RootPart.CFrame = t.Character.HumanoidRootPart.CFrame * CFrame.new(0,0,4)
        WindUI:Notify({Title="TP", Content="A "..TargetName, Duration=2})
    else WindUI:Notify({Title="Error", Content="Target no disponible", Duration=2}) end
end
local function BringTarget()
    local t = ValidTarget()
    if not t then WindUI:Notify({Title="Bring", Content="Selecciona un jugador", Duration=2}); return end
    if t.Character and t.Character:FindFirstChild("HumanoidRootPart") and RootPart then
        pcall(function() t.Character.HumanoidRootPart.CFrame = RootPart.CFrame * CFrame.new(0,0,3) end)
        WindUI:Notify({Title="Bring", Content=TargetName.." traído a ti", Duration=2})
    else WindUI:Notify({Title="Error", Content="Target no disponible", Duration=2}) end
end
local function FreezeTarget(anchor)
    local t = ValidTarget()
    if t and t.Character then
        local hrp = t.Character:FindFirstChild("HumanoidRootPart")
        if hrp then
            pcall(function() hrp.Anchored = anchor end)
            WindUI:Notify({Title="Target", Content=anchor and "Congelado" or "Descongelado", Duration=2})
            return
        end
    end
    WindUI:Notify({Title="Error", Content="Target no disponible", Duration=2})
end
local function RefreshTargetDropdowns()
    pcall(function() if TargetDropdown then TargetDropdown:Refresh(GetPlayerNames(false)) end end)
    pcall(function() if GameTargetDropdown then GameTargetDropdown:Refresh(GetPlayerNames(false)) end end)
    pcall(function() if SpectateDropdown then SpectateDropdown:Refresh(GetPlayerNames(true)) end end)
    pcall(function() if TPPlayerDropdown then TPPlayerDropdown:Refresh(GetPlayerNames(false)) end end)
    pcall(function() if MorphDropdown then MorphDropdown:Refresh(GetPlayerNames(false)) end end)
    WindUI:Notify({Title="Listas", Content="Jugadores recargados", Duration=2})
end

-- 💾 CONFIGURACIÓN PERSISTENTE
local ConfigFileName = "DENJI_ALEX_Config.json"
local Saved = {}
local function Get(key, def)
    if Saved[key] ~= nil then return Saved[key] end
    return def
end

local function GuardarConfiguracion(silent)
    local Config = {
        FlashAttackEnabled = FlashAttackEnabled, FlashMultiplier = FlashMultiplier,
        TPWalkEnabled = TPWalkEnabled, TPWalkSpeed = TPWalkSpeed,
        NoFrictionEnabled = NoFrictionEnabled, InvisibleEnabled = InvisibleEnabled,
        FlyEnabled = FlyEnabled, FlySpeed = FlySpeed,
        NoclipEnabled = NoclipEnabled, InfJumpEnabled = InfJumpEnabled,
        WalkSpeed = Humanoid and Humanoid.WalkSpeed or 16,
        JumpPower = Humanoid and Humanoid.JumpPower or 50,
        GravityScale = Humanoid and Humanoid.GravityScale or 1,
        AutoWalkEnabled = AutoWalkEnabled,
        FullbrightEnabled = FullbrightEnabled, ESPEnabled = ESPEnabled, FpsBoostEnabled = FpsBoostEnabled,
        FOV = (function() local ok,v=pcall(function() return workspace.CurrentCamera.FieldOfView end); return ok and v or 70 end)(),
        ClockTime = Lighting.ClockTime,
        AnimationSpeed = Humanoid and Humanoid.AnimationSpeed or 1,
        BodyScale = Humanoid and Humanoid.BodyHeightScale or 1,
        AutoRejoinEnabled = AutoRejoinEnabled, AntiVoidEnabled = AntiVoidEnabled, AntiRagdollEnabled = AntiRagdollEnabled,
        GodModeEnabled = GodModeEnabled, InstantRespawnEnabled = InstantRespawnEnabled, FollowPlayerEnabled = FollowPlayerEnabled,
        ClickTPEnabled = ClickTPEnabled, AutoJumpEnabled = AutoJumpEnabled, WalkOnWaterEnabled = WalkOnWaterEnabled,
        SpinBotEnabled = SpinBotEnabled, FreezePositionEnabled = FreezePositionEnabled, FallSpeedCap = FallSpeedCap,
        ChatSpamEnabled = ChatSpamEnabled, DanceSpamEnabled = DanceSpamEnabled, RainbowEnabled = RainbowEnabled,
        DrunkCamEnabled = DrunkCamEnabled, PlayDeadEnabled = PlayDeadEnabled, ChatEchoEnabled = ChatEchoEnabled,
        BigHeadEnabled = BigHeadEnabled, ScreenShakeEnabled = ScreenShakeEnabled, SpamText = SpamText,
        AntiAFKEnabled = AntiAFKEnabled, NoPushEnabled = NoPushEnabled, NoKnockbackEnabled = NoKnockbackEnabled,
        SitProtectorEnabled = SitProtectorEnabled,
        AutoClickerEnabled = AutoClickerEnabled, AutoClickerCPS = AutoClickerCPS,
        NoFallDamageEnabled = NoFallDamageEnabled,
    }
    pcall(function()
        local Http = game:GetService("HttpService")
        writefile(ConfigFileName, Http:JSONEncode(Config))
        if not silent then WindUI:Notify({Title="Configuración", Content="Guardada correctamente", Duration=3}) end
    end)
end

local function CargarConfiguracion()
    pcall(function()
        if not isfile(ConfigFileName) then return end
        local Http = game:GetService("HttpService")
        local Config = Http:JSONDecode(readfile(ConfigFileName))
        if Config then Saved = Config end
        if Saved.FlashMultiplier then FlashMultiplier = Saved.FlashMultiplier end
        if Saved.TPWalkSpeed then TPWalkSpeed = Saved.TPWalkSpeed end
        if Saved.FlySpeed then FlySpeed = Saved.FlySpeed end
        if Saved.FallSpeedCap then FallSpeedCap = Saved.FallSpeedCap end
        if Saved.SpamText then SpamText = Saved.SpamText end
        if Saved.AutoClickerCPS then AutoClickerCPS = Saved.AutoClickerCPS end
        WindUI:Notify({Title="Configuración", Content="Cargada correctamente", Duration=3})
    end)
end

local function AplicarConfiguracion()
    pcall(function()
        if not next(Saved) then return end
        if Humanoid then
            if Saved.WalkSpeed then Humanoid.WalkSpeed = Saved.WalkSpeed end
            if Saved.JumpPower then Humanoid.JumpPower = Saved.JumpPower end
            if Saved.GravityScale then Humanoid.GravityScale = Saved.GravityScale end
            if Saved.AnimationSpeed then pcall(function() Humanoid.AnimationSpeed = Saved.AnimationSpeed end) end
            if Saved.BodyScale then
                pcall(function()
                    Humanoid.BodyHeightScale=Saved.BodyScale; Humanoid.BodyWidthScale=Saved.BodyScale; Humanoid.BodyDepthScale=Saved.BodyScale
                    if Humanoid.HeadScale then Humanoid.HeadScale=Saved.BodyScale end
                end)
            end
        end
        if Saved.FOV then pcall(function() workspace.CurrentCamera.FieldOfView = Saved.FOV end) end
        if Saved.ClockTime then Lighting.ClockTime = Saved.ClockTime end
        if Saved.InvisibleEnabled and Character then
            for _,v in pairs(Character:GetDescendants()) do if v:IsA("BasePart") then v.LocalTransparencyModifier=1 end end
            InvisibleEnabled = true
        end
        if Saved.FlashAttackEnabled then SetFlashAttack(true) end
        if Saved.TPWalkEnabled then SetTPWalk(true) end
        if Saved.FlyEnabled then StartFly() end
        if Saved.NoclipEnabled then SetNoclip(true) end
        if Saved.InfJumpEnabled then SetInfJump(true) end
        if Saved.AutoWalkEnabled then SetAutoWalk(true) end
        if Saved.FullbrightEnabled then SetFullbright(true) end
        if Saved.ESPEnabled then SetESP(true) end
        if Saved.FpsBoostEnabled then SetFpsBoost(true) end
        if Saved.ClickTPEnabled then SetClickTP(true) end
        if Saved.SpinBotEnabled then SetSpinBot(true) end
        if Saved.ChatSpamEnabled then SetChatSpam(true) end
        if Saved.DanceSpamEnabled then SetDanceSpam(true) end
        if Saved.RainbowEnabled then SetRainbow(true) end
        if Saved.DrunkCamEnabled then SetDrunkCam(true) end
        if Saved.PlayDeadEnabled then SetPlayDead(true) end
        if Saved.ChatEchoEnabled then SetChatEcho(true) end
        if Saved.BigHeadEnabled then SetBigHead(true) end
        if Saved.ScreenShakeEnabled then SetScreenShake(true) end
        if Saved.SitProtectorEnabled then SetSitProtector(true) end
        if Saved.AutoClickerEnabled then SetAutoClicker(true) end
        if Saved.NoFallDamageEnabled then SetNoFallDamage(true) end
        if Saved.AntiAFKEnabled then SetAntiAFK(true) end
        if Saved.FreezePositionEnabled and RootPart then pcall(function() RootPart.Anchored=true end); FreezePositionEnabled=true end
        if Saved.NoFrictionEnabled~=nil then NoFrictionEnabled=Saved.NoFrictionEnabled end
        if Saved.NoPushEnabled~=nil then NoPushEnabled=Saved.NoPushEnabled end
        if Saved.NoKnockbackEnabled~=nil then NoKnockbackEnabled=Saved.NoKnockbackEnabled end
        if Saved.AntiVoidEnabled~=nil then AntiVoidEnabled=Saved.AntiVoidEnabled end
        if Saved.AntiRagdollEnabled~=nil then AntiRagdollEnabled=Saved.AntiRagdollEnabled end
        if Saved.GodModeEnabled~=nil then GodModeEnabled=Saved.GodModeEnabled end
        if Saved.AutoRejoinEnabled~=nil then AutoRejoinEnabled=Saved.AutoRejoinEnabled end
        if Saved.InstantRespawnEnabled~=nil then InstantRespawnEnabled=Saved.InstantRespawnEnabled end
        if Saved.FollowPlayerEnabled~=nil then FollowPlayerEnabled=Saved.FollowPlayerEnabled end
        if Saved.AutoJumpEnabled~=nil then AutoJumpEnabled=Saved.AutoJumpEnabled end
        if Saved.WalkOnWaterEnabled~=nil then WalkOnWaterEnabled=Saved.WalkOnWaterEnabled end
    end)
end

CargarConfiguracion()

local AvatarUrl
pcall(function() AvatarUrl=Players:GetUserThumbnailAsync(UserId, Enum.ThumbnailType.HeadShot, Enum.ThumbnailSize.Size150x150) end)
if not AvatarUrl or AvatarUrl=="" then AvatarUrl="rbxthumb://type=AvatarHeadShot&id="..UserId.."&w=150&h=150" end

local Window = WindUI:CreateWindow({
    Title="DENJI•ALEX", Icon="sword", Author="DENJI•ALEX", Folder="DENJI•ALEX",
    Size=UDim2.fromOffset(600,540), MinSize=Vector2.new(520,420), MaxSize=Vector2.new(850,680),
    Transparent=true, Theme="Dark", Resizable=true, SideBarWidth=160,
    Background="rbxassetid://118321081493035", BackgroundImageTransparency=0.35, HideSearchBar=true,
    OpenButton={Title="DENJI•ALEX", Icon="sword", Enabled=true, Draggable=true, OnlyMobile=false, CornerRadius=UDim.new(1,0), StrokeThickness=2, Scale=1},
})

-- 1. PLAYER
local PlayerTab = Window:Tab({Title="Player", Icon="user"})
PlayerTab:Image({Image=AvatarUrl, ImageSize=100, ImageColor=Color3.fromRGB(255,255,255)})
PlayerTab:Paragraph({Title=DisplayName, Desc="@"..PlayerName.."  |  ID: "..UserId, Image="user", ImageSize=18})
PlayerTab:Space({Size=8})
local StatsGroup = PlayerTab:Group({})
local InfoSection = StatsGroup:Section({Title="Cuenta", Box=true, BoxBorder=true, Opened=true})
InfoSection:Paragraph({Title="Jugadores", Desc=#Players:GetPlayers().." / "..Players.MaxPlayers, Image="users", ImageSize=14}); InfoSection:Space({Size=4})
InfoSection:Paragraph({Title="Display", Desc=DisplayName, Image="user", ImageSize=14}); InfoSection:Space({Size=4})
InfoSection:Paragraph({Title="Usuario", Desc="@"..PlayerName, Image="at-sign", ImageSize=14}); InfoSection:Space({Size=4})
InfoSection:Paragraph({Title="User ID", Desc=tostring(UserId), Image="hash", ImageSize=14})
StatsGroup:Space({Size=10})
local PerfSection = StatsGroup:Section({Title="Rendimiento", Box=true, BoxBorder=true, Opened=true})
local StatsParagraph = PerfSection:Paragraph({Title="En Tiempo Real", Desc="Ping: Cargando...\nFPS: Cargando...\nMemoria: Cargando...", Image="activity", ImageSize=14})
task.spawn(function()
    while task.wait(1) do
        local ps, Ping = pcall(function() return math.floor(game:GetService("Stats").Network.ServerStatsItem["Data Ping"]:GetValue()) end)
        local FPS = math.floor(1/RunService.Heartbeat:Wait())
        local Memory = math.floor(game:GetService("Stats"):GetTotalMemoryUsageMb())
        if StatsParagraph and StatsParagraph.SetDesc then StatsParagraph:SetDesc("Ping: "..(ps and (Ping.." ms") or "N/A").."\nFPS: "..FPS.."\nMemoria: "..Memory.." MB") end
    end
end)

-- 2. MAIN
local MainTab = Window:Tab({Title="Main", Icon="home"})
MainTab:Section({Title="Funciones Principales", TextSize=20}); MainTab:Space({Size=6})
MainTab:Section({Title="⚔️ Ataque Rápido", TextSize=18}); MainTab:Space({Size=6})
local FlashRow = MainTab:Group({})
FlashRow:Toggle({Title="Activar Ataque Rápido", Def=Get("FlashAttackEnabled", false), Callback=SetFlashAttack})
FlashRow:Space({Size=8})
FlashRow:Slider({Title="Multiplicador", Step=1, Value={Min=1,Max=30,Default=Get("FlashMultiplier", 5)}, Callback=function(v) FlashMultiplier = v end})
MainTab:Space({Size=12})
local MainRow1 = MainTab:Group({})
MainRow1:Button({Title="Teleport a Ti", Icon="map-pin", Justify="Center", Callback=function() local m=LocalPlayer:GetMouse(); if RootPart then RootPart.CFrame=CFrame.new(m.Hit.Position+Vector3.new(0,3,0)) end end})
MainRow1:Space({Size=8})
MainRow1:Toggle({Title="TPWalk (Bypass)", Def=Get("TPWalkEnabled", false), Callback=SetTPWalk})
MainTab:Space({Size=8})
MainTab:Slider({Title="Velocidad TPWalk", Step=0.05, Value={Min=0.01, Max=20, Default=Get("TPWalkSpeed", 0.30)}, Callback=function(v) TPWalkSpeed = v end})
MainTab:Space({Size=8})
local MainRow2 = MainTab:Group({})
MainRow2:Toggle({Title="Salto Alto", Def=false, Callback=function(s) if Humanoid then Humanoid.JumpPower=s and 120 or 50 end end})
MainRow2:Space({Size=8})
MainRow2:Toggle({Title="Salto Infinito", Def=Get("InfJumpEnabled", false), Callback=function(s) SetInfJump(s) end})
MainTab:Space({Size=8})
local MainRow3 = MainTab:Group({})
MainRow3:Toggle({Title="Sin Fricción", Def=Get("NoFrictionEnabled", false), Callback=function(s) NoFrictionEnabled=s end})
MainRow3:Space({Size=8})
MainRow3:Toggle({Title="Sin Gravedad", Def=false, Callback=function(s) if Humanoid then Humanoid.GravityScale=s and 0 or 1 end end})
MainTab:Space({Size=8})
local MainRow4 = MainTab:Group({})
MainRow4:Toggle({Title="Invisible", Def=Get("InvisibleEnabled", false), Callback=function(s) InvisibleEnabled=s; if Character then for _,v in pairs(Character:GetDescendants()) do if v:IsA("BasePart") then v.LocalTransparencyModifier=s and 1 or 0 end end end end})
MainTab:Space({Size=12})
MainTab:Section({Title="Ajustes de Movimiento", TextSize=18}); MainTab:Space({Size=6})
local MainSliders = MainTab:Section({Title="Sliders Rápidos", Box=true, BoxBorder=true, Opened=true})
MainSliders:Slider({Title="WalkSpeed", Step=1, Value={Min=16,Max=250,Default=Get("WalkSpeed", 16)}, Callback=function(v) if Humanoid then Humanoid.WalkSpeed=v end end}); MainSliders:Space({Size=6})
MainSliders:Slider({Title="JumpPower", Step=1, Value={Min=50,Max=350,Default=Get("JumpPower", 50)}, Callback=function(v) if Humanoid then Humanoid.JumpPower=v end end}); MainSliders:Space({Size=6})
MainSliders:Slider({Title="Gravedad", Step=0.1, Value={Min=0,Max=2,Default=Get("GravityScale", 1)}, Callback=function(v) if Humanoid then Humanoid.GravityScale=v end end}); MainSliders:Space({Size=6})
MainSliders:Toggle({Title="Auto-Caminar (Hacia Adelante)", Def=Get("AutoWalkEnabled", false), Callback=SetAutoWalk})
MainTab:Space({Size=12})
MainTab:Section({Title="Acciones Rápidas", TextSize=18}); MainTab:Space({Size=6})
local MainA1 = MainTab:Group({})
MainA1:Button({Title="Server Hop", Icon="shuffle", Justify="Center", Callback=ServerHop}); MainA1:Space({Size=8})
MainA1:Button({Title="Rejoin (Mismo Server)", Icon="refresh-cw", Justify="Center", Callback=function() TeleportService:TeleportToPlaceInstance(game.PlaceId, game.JobId) end})
MainTab:Space({Size=8})
local MainA2 = MainTab:Group({})
MainA2:Button({Title="Copiar Coordenadas", Icon="clipboard", Justify="Center", Callback=function() if RootPart then local p=RootPart.Position; setclipboard(math.floor(p.X)..", "..math.floor(p.Y)..", "..math.floor(p.Z)); WindUI:Notify({Title="Copiado", Content="Coordenadas copiadas", Duration=2}) end end})
MainA2:Space({Size=8})
MainA2:Button({Title="TP desde Portapapeles", Icon="map-pin", Justify="Center", Callback=function() local clip=""; pcall(function() clip=getclipboard() end); if not clip or clip=="" then WindUI:Notify({Title="Error", Content="Portapapeles vacío", Duration=2}) else TeleportToCoords(clip) end end})
MainTab:Space({Size=8})
local MainA3 = MainTab:Group({})
MainA3:Button({Title="Volver al Spawn", Icon="home", Justify="Center", Callback=function() if RootPart then RootPart.CFrame=SpawnCFrame; RootPart.Velocity=Vector3.new(0,0,0); WindUI:Notify({Title="Spawn", Content="Volviste al spawn", Duration=2}) end end})
MainA3:Space({Size=8})
MainA3:Button({Title="Reiniciar Personaje", Icon="refresh-cw", Justify="Center", Callback=function() if Character and Humanoid then Humanoid.Health=0 end end})
MainTab:Space({Size=12})
MainTab:Section({Title="📦 Scripts Externos", TextSize=18}); MainTab:Space({Size=6})
local ExtScripts = MainTab:Section({Title="", Box=true, BoxBorder=true, Opened=true})
ExtScripts:Button({Title="Hitbox Boys", Desc="Ejecutar script", Icon="play", Justify="Left", Callback=function()
    pcall(function() loadstring(game:HttpGet("https://pastebin.com/raw/4vL0qwVd"))(); WindUI:Notify({Title="Hitbox Boys", Content="Script ejecutado", Duration=3}) end)
end})
ExtScripts:Space({Size=10})
ExtScripts:Button({Title="Hitbox Girls", Desc="Ejecutar script", Icon="play", Justify="Left", Callback=function()
    pcall(function() loadstring(game:HttpGet("https://pastebin.com/raw/x9ivUUsh"))(); WindUI:Notify({Title="Hitbox Girls", Content="Script ejecutado", Duration=3}) end)
end})

-- 3. MIS SCRIPTS
local MisScriptsTab = Window:Tab({Title="Mis Scripts", Icon="folder"})
MisScriptsTab:Section({Title="Scripts Guardados", TextSize=20}); MisScriptsTab:Space({Size=6})
local ScriptsSection = MisScriptsTab:Section({Title="", Box=true, BoxBorder=true, Opened=true})
ScriptsSection:Button({Title="AY1Amikas", Desc="Ejecutar script", Icon="play", Justify="Left", Callback=function()
    pcall(function() loadstring(game:HttpGet("https://raw.githubusercontent.com/alex001xx/AY1AniChoco/refs/heads/main/README.md"))(); WindUI:Notify({Title="AY1Amikas", Content="Script ejecutado", Duration=3}) end)
end})
ScriptsSection:Space({Size=10})
ScriptsSection:Toggle({Title="Escudo", Desc="Sit siempre + Anti-abrazo", Def=Get("SitProtectorEnabled", false), Callback=SetSitProtector})
MisScriptsTab:Space({Size=12})

-- 4. TERGET (objetivo/jugador)
local TergetTab = Window:Tab({Title="Terget", Icon="target"})
TergetTab:Section({Title="Jugador Objetivo", TextSize=20}); TergetTab:Space({Size=6})
TargetDropdown = TergetTab:Dropdown({Title="Seleccionar Target", Values=GetPlayerNames(false), Value=1, Callback=function(s) TargetName=s end})
TergetTab:Space({Size=8})
TargetInfoParagraph = TergetTab:Paragraph({Title="Info del Target", Desc="Ninguno", Image="user", ImageSize=14})
TergetTab:Space({Size=8})
TergetTab:Section({Title="Acciones sobre el Target", TextSize=18}); TergetTab:Space({Size=6})
local TAct = TergetTab:Section({Title="", Box=true, BoxBorder=true, Opened=true})
local TA1 = TAct:Group({})
TA1:Button({Title="Focus (Cámara)", Icon="eye", Justify="Center", Callback=FocusTarget}); TA1:Space({Size=8})
TA1:Button({Title="Detener Focus", Icon="eye-off", Justify="Center", Callback=StopFocus})
TAct:Space({Size=6})
local TA2 = TAct:Group({})
TA2:Button({Title="TP al Target", Icon="map-pin", Justify="Center", Callback=TPToTarget}); TA2:Space({Size=8})
TA2:Button({Title="Traer Target (Bring)", Icon="arrow-down", Justify="Center", Callback=BringTarget})
TAct:Space({Size=6})
TAct:Toggle({Title="Seguir Target (Follow)", Def=false, Callback=function(s) if TargetName then TPPlayerName=TargetName end; FollowPlayerEnabled=s end})
TAct:Space({Size=6})
local TA3 = TAct:Group({})
TA3:Button({Title="Congelar Target", Icon="lock", Justify="Center", Callback=function() FreezeTarget(true) end}); TA3:Space({Size=8})
TA3:Button({Title="Descongelar Target", Icon="unlock", Justify="Center", Callback=function() FreezeTarget(false) end})
TAct:Space({Size=6})
TAct:Button({Title="Copiar UserID del Target", Icon="clipboard", Justify="Center", Callback=function()
    local t = ValidTarget()
    if t then setclipboard(tostring(t.UserId)); WindUI:Notify({Title="Copiado", Content="UserID de "..t.Name, Duration=2})
    else WindUI:Notify({Title="Error", Content="Selecciona un target", Duration=2}) end
end})
TergetTab:Space({Size=8})
TergetTab:Button({Title="Recargar Lista de Jugadores", Icon="refresh-cw", Justify="Center", Callback=RefreshTargetDropdowns})
TergetTab:Space({Size=12})

-- 5. GAME (FUNCIONES UNIVERSALES)
local GameTab = Window:Tab({Title="Game", Icon="gamepad-2"})
GameTab:Section({Title="Funciones Universales", TextSize=20}); GameTab:Space({Size=6})

GameTab:Section({Title="🎯 Target (Jugador)", TextSize=18}); GameTab:Space({Size=6})
local GTgt = GameTab:Section({Title="", Box=true, BoxBorder=true, Opened=true})
GameTargetDropdown = GTgt:Dropdown({Title="Seleccionar Target", Values=GetPlayerNames(false), Value=1, Callback=function(s) TargetName=s end})
GTgt:Space({Size=6})
local GT1 = GTgt:Group({})
GT1:Button({Title="Focus", Icon="eye", Justify="Center", Callback=FocusTarget}); GT1:Space({Size=8})
GT1:Button({Title="TP al Target", Icon="map-pin", Justify="Center", Callback=TPToTarget})
GTgt:Space({Size=6})
local GT2 = GTgt:Group({})
GT2:Button({Title="Traer (Bring)", Icon="arrow-down", Justify="Center", Callback=BringTarget}); GT2:Space({Size=8})
GT2:Button({Title="TP All a Mí", Icon="users", Justify="Center", Callback=TPAllToMe})
GameTab:Space({Size=10})

GameTab:Section({Title="🚀 Movimiento", TextSize=18}); GameTab:Space({Size=6})
local GMov = GameTab:Section({Title="", Box=true, BoxBorder=true, Opened=true})
GMov:Toggle({Title="Fly (Volar)", Def=Get("FlyEnabled", false), Callback=function(s) if s then StartFly() else StopFly() end end}); GMov:Space({Size=6})
GMov:Toggle({Title="Noclip", Def=Get("NoclipEnabled", false), Callback=SetNoclip}); GMov:Space({Size=6})
GMov:Toggle({Title="Salto Infinito", Def=Get("InfJumpEnabled", false), Callback=SetInfJump}); GMov:Space({Size=6})
GMov:Toggle({Title="Click TP (Clic Der.)", Def=Get("ClickTPEnabled", false), Callback=SetClickTP}); GMov:Space({Size=6})
GMov:Toggle({Title="Auto-Jump (Bunny Hop)", Def=Get("AutoJumpEnabled", false), Callback=function(s) AutoJumpEnabled=s end}); GMov:Space({Size=6})
GMov:Toggle({Title="Walk on Water", Def=Get("WalkOnWaterEnabled", false), Callback=function(s) WalkOnWaterEnabled=s end}); GMov:Space({Size=6})
GMov:Toggle({Title="Spin Bot", Def=Get("SpinBotEnabled", false), Callback=SetSpinBot}); GMov:Space({Size=6})
GMov:Toggle({Title="Freeze Position", Def=Get("FreezePositionEnabled", false), Callback=SetFreeze}); GMov:Space({Size=6})
GMov:Slider({Title="WalkSpeed", Step=1, Value={Min=16,Max=250,Default=Get("WalkSpeed",16)}, Callback=function(v) if Humanoid then Humanoid.WalkSpeed=v end end}); GMov:Space({Size=6})
GMov:Slider({Title="JumpPower", Step=1, Value={Min=50,Max=350,Default=Get("JumpPower",50)}, Callback=function(v) if Humanoid then Humanoid.JumpPower=v end end}); GMov:Space({Size=6})
GMov:Slider({Title="Gravedad", Step=0.1, Value={Min=0,Max=2,Default=Get("GravityScale",1)}, Callback=function(v) if Humanoid then Humanoid.GravityScale=v end end})
GameTab:Space({Size=10})

GameTab:Section({Title="👁️ Visual", TextSize=18}); GameTab:Space({Size=6})
local GVis = GameTab:Section({Title="", Box=true, BoxBorder=true, Opened=true})
GVis:Toggle({Title="ESP Jugadores", Def=Get("ESPEnabled", false), Callback=SetESP}); GVis:Space({Size=6})
GVis:Toggle({Title="Fullbright", Def=Get("FullbrightEnabled", false), Callback=SetFullbright}); GVis:Space({Size=6})
GVis:Toggle({Title="FPS Boost", Def=Get("FpsBoostEnabled", false), Callback=SetFpsBoost}); GVis:Space({Size=6})
GVis:Slider({Title="FOV de Cámara", Step=1, Value={Min=60,Max=120,Default=Get("FOV",70)}, Callback=function(v) pcall(function() workspace.CurrentCamera.FieldOfView=v end) end}); GVis:Space({Size=6})
local GVisB = GVis:Group({})
GVisB:Button({Title="Desbloquear Zoom", Icon="zoom-in", Justify="Center", Callback=function() pcall(function() workspace.CurrentCamera.CameraMaxZoomDistance=1000; workspace.CurrentCamera.CameraMinZoomDistance=0.5 end); WindUI:Notify({Title="Zoom", Content="Desbloqueado", Duration=2}) end})
GVisB:Space({Size=8})
GVisB:Button({Title="Eliminar Partículas", Icon="trash-2", Justify="Center", Callback=RemoveParticles})
GameTab:Space({Size=10})

GameTab:Section({Title="🛡️ Protección", TextSize=18}); GameTab:Space({Size=6})
local GProt = GameTab:Section({Title="", Box=true, BoxBorder=true, Opened=true})
GProt:Toggle({Title="God Mode (Local)", Def=Get("GodModeEnabled", false), Callback=function(s) GodModeEnabled=s end}); GProt:Space({Size=6})
GProt:Toggle({Title="Anti-Void", Def=Get("AntiVoidEnabled", false), Callback=function(s) AntiVoidEnabled=s end}); GProt:Space({Size=6})
GProt:Toggle({Title="Anti-Ragdoll", Def=Get("AntiRagdollEnabled", false), Callback=function(s) AntiRagdollEnabled=s end}); GProt:Space({Size=6})
GProt:Toggle({Title="Anti-AFK (Real)", Def=Get("AntiAFKEnabled", false), Callback=SetAntiAFK}); GProt:Space({Size=6})
GProt:Toggle({Title="No Fall Damage", Def=Get("NoFallDamageEnabled", false), Callback=SetNoFallDamage}); GProt:Space({Size=6})
GProt:Toggle({Title="Auto-Respawn Instantáneo", Def=Get("InstantRespawnEnabled", false), Callback=function(s) InstantRespawnEnabled=s end}); GProt:Space({Size=6})
GProt:Toggle({Title="Auto-Rejoin al Morir", Def=Get("AutoRejoinEnabled", false), Callback=function(s) AutoRejoinEnabled=s end})
GameTab:Space({Size=10})

GameTab:Section({Title="🔧 Utilidades", TextSize=18}); GameTab:Space({Size=6})
local GUtil = GameTab:Section({Title="", Box=true, BoxBorder=true, Opened=true})
GUtil:Toggle({Title="Auto-Clicker", Def=Get("AutoClickerEnabled", false), Callback=SetAutoClicker}); GUtil:Space({Size=6})
GUtil:Slider({Title="CPS del Auto-Clicker", Step=1, Value={Min=1,Max=20,Default=Get("AutoClickerCPS",10)}, Callback=function(v) AutoClickerCPS=v end}); GUtil:Space({Size=8})
local GU1 = GUtil:Group({})
GU1:Button({Title="Rejoin", Icon="refresh-cw", Justify="Center", Callback=function() TeleportService:TeleportToPlaceInstance(game.PlaceId, game.JobId) end}); GU1:Space({Size=8})
GU1:Button({Title="Server Hop", Icon="shuffle", Justify="Center", Callback=ServerHop})
GUtil:Space({Size=6})
local GU2 = GUtil:Group({})
GU2:Button({Title="Copiar JobID", Icon="clipboard", Justify="Center", Callback=function() setclipboard(tostring(game.JobId)); WindUI:Notify({Title="Copiado", Content="JobID copiado", Duration=2}) end}); GU2:Space({Size=8})
GU2:Button({Title="Copiar Link del Juego", Icon="link", Justify="Center", Callback=function() setclipboard("https://www.roblox.com/games/"..tostring(game.PlaceId)); WindUI:Notify({Title="Copiado", Content="Link copiado", Duration=2}) end})
GameTab:Space({Size=10})

GameTab:Section({Title="📦 Scripts Universales", TextSize=18}); GameTab:Space({Size=6})
local GScr = GameTab:Section({Title="", Box=true, BoxBorder=true, Opened=true})
GScr:Button({Title="Infinite Yield (Admin Universal)", Desc="Ejecutar", Icon="play", Justify="Left", Callback=function()
    pcall(function() loadstring(game:HttpGet("https://raw.githubusercontent.com/EdgeIY/infiniteyield/master/source"))(); WindUI:Notify({Title="Infinite Yield", Content="Script ejecutado", Duration=3}) end)
end})
GScr:Space({Size=10})
GScr:Button({Title="Dex Explorer", Desc="Ejecutar", Icon="play", Justify="Left", Callback=function()
    pcall(function() loadstring(game:HttpGet("https://raw.githubusercontent.com/infyiff/backup/main/dex.lua"))(); WindUI:Notify({Title="Dex Explorer", Content="Script ejecutado", Duration=3}) end)
end})
GameTab:Space({Size=12})

-- 6. SERVIDORES
local ServidoresTab = Window:Tab({Title="Servidores", Icon="globe"})
ServidoresTab:Section({Title="Información Actual", TextSize=18}); ServidoresTab:Space({Size=6})
local ServInfoRow = ServidoresTab:Group({})
ServInfoRow:Paragraph({Title="Jugadores", Desc=#Players:GetPlayers().." / "..Players.MaxPlayers, Image="users", ImageSize=14}); ServInfoRow:Space({Size=8})
ServInfoRow:Paragraph({Title="Place ID", Desc=tostring(game.PlaceId), Image="hash", ImageSize=14}); ServInfoRow:Space({Size=8})
ServInfoRow:Paragraph({Title="Job ID", Desc=string.sub(tostring(game.JobId),1,12).."...", Image="clipboard", ImageSize=14})
ServidoresTab:Space({Size=10})
local ServActionRow = ServidoresTab:Group({})
ServActionRow:Button({Title="Reingresar", Icon="refresh-cw", Justify="Center", Callback=function() TeleportService:TeleportToPlaceInstance(game.PlaceId, game.JobId) end}); ServActionRow:Space({Size=8})
ServActionRow:Button({Title="Copiar JobID", Icon="clipboard", Justify="Center", Callback=function() setclipboard(tostring(game.JobId)); WindUI:Notify({Title="Copiado", Content="JobID copiado", Duration=2}) end})
ServidoresTab:Space({Size=12})
ServidoresTab:Section({Title="Servidores Disponibles", TextSize=18}); ServidoresTab:Space({Size=6})
local PublicServersSection = ServidoresTab:Section({Title="", Box=true, BoxBorder=true, Opened=true})
PublicServersSection:Paragraph({Title="Cargando servidores...", Desc="Espera un momento...", Image="loading", ImageSize=16})
local function LoadPublicServers()
    PublicServersSection:Clear()
    PublicServersSection:Paragraph({Title="Cargando servidores...", Desc="Espera un momento...", Image="loading", ImageSize=16})
    task.spawn(function()
        local ok, err = pcall(function()
            local Http=game:GetService("HttpService")
            local res=Http:GetAsync("https://games.roproxy.com/v1/games/"..game.PlaceId.."/servers/Public?sortOrder=Asc&limit=100")
            local response=Http:JSONDecode(res)
            PublicServersSection:Clear()
            if not response or not response.data or #response.data==0 then
                PublicServersSection:Paragraph({Title="Sin servidores", Desc="Intenta recargar", Image="info", ImageSize=16}); return
            end
            PublicServersSection:Paragraph({Title="Encontrados: "..#response.data.." servidores", Desc="Presiona TP para unirte", Image="globe", ImageSize=14})
            PublicServersSection:Space({Size=6})
            for i=1,#response.data,2 do
                local g=PublicServersSection:Group({})
                local s1=response.data[i]; local p1=s1.playing or 0; local m1=s1.maxPlayers or 0
                g:Button({Title="Servidor "..i, Desc=p1.."/"..m1, Justify="Left", Callback=function() TeleportService:TeleportToPlaceInstance(game.PlaceId, s1.id) end})
                g:Button({Title="TP", Icon="send", Justify="Center", Callback=function() TeleportService:TeleportToPlaceInstance(game.PlaceId, s1.id) end})
                if response.data[i+1] then
                    g:Space({Size=8})
                    local s2=response.data[i+1]; local p2=s2.playing or 0; local m2=s2.maxPlayers or 0
                    g:Button({Title="Servidor "..(i+1), Desc=p2.."/"..m2, Justify="Left", Callback=function() TeleportService:TeleportToPlaceInstance(game.PlaceId, s2.id) end})
                    g:Button({Title="TP", Icon="send", Justify="Center", Callback=function() TeleportService:TeleportToPlaceInstance(game.PlaceId, s2.id) end})
                end
                PublicServersSection:Space({Size=6})
            end
        end)
        if not ok then
            PublicServersSection:Clear()
            PublicServersSection:Paragraph({Title="Error al cargar", Desc="El proxy puede estar caído. Recarga.", Image="alert-triangle", ImageSize=16})
            WindUI:Notify({Title="Servidores", Content="Error: "..tostring(err):sub(1,60), Duration=4})
        end
    end)
end
ServidoresTab:Button({Title="Recargar Servidores Públicos", Icon="refresh-cw", Justify="Center", Callback=LoadPublicServers})
ServidoresTab:Space({Size=12})
ServidoresTab:Section({Title="Servidores de Amigos", TextSize=18}); ServidoresTab:Space({Size=6})
local FriendsServersSection = ServidoresTab:Section({Title="", Box=true, BoxBorder=true, Opened=true})
FriendsServersSection:Paragraph({Title="Cargando amigos...", Desc="Espera un momento...", Image="loading", ImageSize=16})
local function JoinFriend(fid, fname)
    task.spawn(function()
        local ok=pcall(function()
            local Http=game:GetService("HttpService")
            local res=Http:RequestAsync({Url="https://presence.roproxy.com/v1/presence/users", Method="POST", Headers={["Content-Type"]="application/json"}, Body=Http:JSONEncode({userIds={tonumber(fid)}})})
            local data=Http:JSONDecode(res.Body)
            local pres=data and data.userPresences and data.userPresences[1]
            if pres and pres.userPresenceType==2 and pres.placeId and pres.gameId then
                WindUI:Notify({Title="Uniéndose", Content="A "..fname, Duration=3})
                TeleportService:TeleportToPlaceInstance(pres.placeId, pres.gameId)
            else WindUI:Notify({Title="No disponible", Content=fname.." no está en un juego", Duration=3}) end
        end)
        if not ok then WindUI:Notify({Title="Error", Content="No se pudo obtener la sesión del amigo", Duration=3}) end
    end)
end
local function LoadFriendsServers()
    FriendsServersSection:Clear()
    FriendsServersSection:Paragraph({Title="Cargando amigos...", Desc="Espera un momento...", Image="loading", ImageSize=16})
    task.spawn(function()
        local ok, err = pcall(function()
            local Http=game:GetService("HttpService")
            local res=Http:GetAsync("https://friends.roproxy.com/v1/users/"..UserId.."/friends")
            local fd=Http:JSONDecode(res)
            FriendsServersSection:Clear()
            if not fd or not fd.data or #fd.data==0 then FriendsServersSection:Paragraph({Title="No tienes amigos", Desc="Agrega amigos para verlos", Image="user-x", ImageSize=16}); return end
            FriendsServersSection:Paragraph({Title="Amigos: "..#fd.data, Desc="Presiona TP para unirte", Image="users", ImageSize=14})
            FriendsServersSection:Space({Size=6})
            for i=1,#fd.data,2 do
                local g=FriendsServersSection:Group({})
                local f1=fd.data[i]; local n1=f1.displayName and f1.displayName~="" and f1.displayName or f1.name
                g:Button({Title=n1, Desc="@"..f1.name, Justify="Left", Callback=function() JoinFriend(f1.id, f1.name) end})
                g:Button({Title="TP", Icon="send", Justify="Center", Callback=function() JoinFriend(f1.id, f1.name) end})
                if fd.data[i+1] then
                    g:Space({Size=8})
                    local f2=fd.data[i+1]; local n2=f2.displayName and f2.displayName~="" and f2.displayName or f2.name
                    g:Button({Title=n2, Desc="@"..f2.name, Justify="Left", Callback=function() JoinFriend(f2.id, f2.name) end})
                    g:Button({Title="TP", Icon="send", Justify="Center", Callback=function() JoinFriend(f2.id, f2.name) end})
                end
                FriendsServersSection:Space({Size=6})
            end
        end)
        if not ok then
            FriendsServersSection:Clear()
            FriendsServersSection:Paragraph({Title="Error al cargar", Desc="El proxy puede estar caído. Recarga.", Image="alert-triangle", ImageSize=16})
            WindUI:Notify({Title="Amigos", Content="Error: "..tostring(err):sub(1,60), Duration=4})
        end
    end)
end
ServidoresTab:Button({Title="Recargar Amigos", Icon="refresh-cw", Justify="Center", Callback=LoadFriendsServers})
ServidoresTab:Space({Size=10})
task.spawn(function() task.wait(1.5); LoadPublicServers(); LoadFriendsServers() end)

-- 7. ESCUDOS
local EscudosTab = Window:Tab({Title="Escudos", Icon="shield"})
EscudosTab:Section({Title="Protección y Defensas", TextSize=20}); EscudosTab:Space({Size=6})
local SR1=EscudosTab:Group({})
SR1:Toggle({Title="Anti-AFK", Def=Get("AntiAFKEnabled", false), Callback=SetAntiAFK}); SR1:Space({Size=8})
SR1:Toggle({Title="Anti-Kick", Def=false, Callback=function(s) pcall(function() StarterGui:SetCore("ResetButtonCallback", not s) end) end})
EscudosTab:Space({Size=8})
local function shieldRow(t1, t2)
    local r=EscudosTab:Group({})
    r:Toggle({Title=t1, Def=false, Callback=function() end}); r:Space({Size=8}); r:Toggle({Title=t2, Def=false, Callback=function() end})
    EscudosTab:Space({Size=8})
end
shieldRow("Anti-Ban","Anti-Detect"); shieldRow("Protección Chat","Protección Vote")
shieldRow("Anti-Report","Anti-Mute"); shieldRow("Anti-Crash","Anti-Freeze")
local SR6=EscudosTab:Group({})
SR6:Toggle({Title="No Empuje", Def=Get("NoPushEnabled", false), Callback=function(s) NoPushEnabled=s end}); SR6:Space({Size=8})
SR6:Toggle({Title="No Retroceso", Def=Get("NoKnockbackEnabled", false), Callback=function(s) NoKnockbackEnabled=s end})

-- 8. CONFIGURACIÓN
local ConfigTab = Window:Tab({Title="Configuración", Icon="settings"})
ConfigTab:Section({Title="Ajustes del Menú", TextSize=20}); ConfigTab:Space({Size=6})
local C1=ConfigTab:Group({})
C1:Button({Title="Cerrar Menú", Icon="x", Justify="Center", Callback=function() Window:Close() end}); C1:Space({Size=8})
C1:Button({Title="Reiniciar", Icon="refresh-cw", Justify="Center", Callback=function() if Character then Humanoid.Health=0 end end})
ConfigTab:Space({Size=8})
local C2=ConfigTab:Group({})
C2:Button({Title="Copiar UserID", Icon="clipboard", Justify="Center", Callback=function() setclipboard(tostring(UserId)); WindUI:Notify({Title="Copiado", Content="UserID copiado", Duration=2}) end}); C2:Space({Size=8})
C2:Button({Title="Copiar Username", Icon="clipboard", Justify="Center", Callback=function() setclipboard("@"..PlayerName); WindUI:Notify({Title="Copiado", Content="Username copiado", Duration=2}) end})
ConfigTab:Space({Size=12})
ConfigTab:Section({Title="Guardado (Auto-Guardado cada 10s)", TextSize=18}); ConfigTab:Space({Size=6})
local CFGRow=ConfigTab:Group({})
CFGRow:Button({Title="Guardar Configuración", Icon="save", Justify="Center", Callback=function() GuardarConfiguracion(false) end})
CFGRow:Space({Size=8})
CFGRow:Button({Title="Cargar Configuración", Icon="refresh-cw", Justify="Center", Callback=function() CargarConfiguracion(); AplicarConfiguracion(); WindUI:Notify({Title="Configuración", Content="Aplicada", Duration=2}) end})

-- 9. HERRAMIENTAS
local H = Window:Tab({Title="Herramientas", Icon="wrench"})
H:Section({Title="Movimiento", TextSize=20}); H:Space({Size=6})
local Mov = H:Section({Title="Controles de Movimiento", Box=true, BoxBorder=true, Opened=true})
Mov:Toggle({Title="Fly (Volar)", Def=Get("FlyEnabled", false), Callback=function(s) if s then StartFly() else StopFly() end end}); Mov:Space({Size=6})
Mov:Slider({Title="Velocidad de Fly", Step=5, Value={Min=10,Max=200,Default=Get("FlySpeed", 60)}, Callback=function(v) FlySpeed=v end}); Mov:Space({Size=6})
Mov:Toggle({Title="Noclip", Def=Get("NoclipEnabled", false), Callback=SetNoclip}); Mov:Space({Size=6})
Mov:Toggle({Title="Salto Infinito", Def=Get("InfJumpEnabled", false), Callback=SetInfJump}); Mov:Space({Size=6})
Mov:Slider({Title="WalkSpeed", Step=1, Value={Min=16,Max=250,Default=Get("WalkSpeed", 16)}, Callback=function(v) if Humanoid then Humanoid.WalkSpeed=v end end}); Mov:Space({Size=6})
Mov:Slider({Title="JumpPower", Step=1, Value={Min=50,Max=350,Default=Get("JumpPower", 50)}, Callback=function(v) if Humanoid then Humanoid.JumpPower=v end end}); Mov:Space({Size=6})
Mov:Slider({Title="Gravedad", Step=0.1, Value={Min=0,Max=2,Default=Get("GravityScale", 1)}, Callback=function(v) if Humanoid then Humanoid.GravityScale=v end end})
H:Space({Size=10})
H:Section({Title="Visual", TextSize=20}); H:Space({Size=6})
local Vis = H:Section({Title="Efectos Visuales", Box=true, BoxBorder=true, Opened=true})
Vis:Toggle({Title="Fullbright", Def=Get("FullbrightEnabled", false), Callback=SetFullbright}); Vis:Space({Size=6})
Vis:Toggle({Title="ESP Jugadores", Def=Get("ESPEnabled", false), Callback=SetESP}); Vis:Space({Size=6})
Vis:Toggle({Title="FPS Boost", Def=Get("FpsBoostEnabled", false), Callback=SetFpsBoost}); Vis:Space({Size=6})
Vis:Slider({Title="FOV de Cámara", Step=1, Value={Min=60,Max=120,Default=Get("FOV", 70)}, Callback=function(v) pcall(function() workspace.CurrentCamera.FieldOfView=v end) end}); Vis:Space({Size=6})
Vis:Button({Title="Desbloquear Zoom", Icon="zoom-in", Justify="Center", Callback=function() pcall(function() workspace.CurrentCamera.CameraMaxZoomDistance=1000; workspace.CurrentCamera.CameraMinZoomDistance=0.5 end); WindUI:Notify({Title="Zoom", Content="Zoom desbloqueado", Duration=2}) end})
H:Space({Size=10})
H:Section({Title="Utilidades", TextSize=20}); H:Space({Size=6})
local Uti = H:Section({Title="Herramientas Universales", Box=true, BoxBorder=true, Opened=true})
Uti:Toggle({Title="Auto-Rejoin al Morir", Def=Get("AutoRejoinEnabled", false), Callback=function(s) AutoRejoinEnabled=s end}); Uti:Space({Size=6})
Uti:Toggle({Title="Anti-Void", Def=Get("AntiVoidEnabled", false), Callback=function(s) AntiVoidEnabled=s end}); Uti:Space({Size=6})
Uti:Toggle({Title="Anti-Ragdoll", Def=Get("AntiRagdollEnabled", false), Callback=function(s) AntiRagdollEnabled=s end}); Uti:Space({Size=8})
SpectateDropdown = Uti:Dropdown({Title="Jugador a Espectear", Values=GetPlayerNames(true), Value=1, Callback=function(s) SpectateName=s end})
Uti:Space({Size=6})
local SpR=Uti:Group({})
SpR:Button({Title="Espectear", Icon="eye", Justify="Center", Callback=function()
    if not SpectateName or SpectateName=="Nadie (detener)" or SpectateName=="No hay jugadores" then pcall(function() workspace.CurrentCamera.CameraSubject=Humanoid end); WindUI:Notify({Title="Espectear", Content="Selecciona un jugador", Duration=2}); return end
    local t=Players:FindFirstChild(SpectateName)
    if t and t.Character and t.Character:FindFirstChild("Humanoid") then workspace.CurrentCamera.CameraSubject=t.Character.Humanoid; WindUI:Notify({Title="Especteando", Content="A "..SpectateName, Duration=2})
    else WindUI:Notify({Title="Error", Content="Jugador no disponible", Duration=2}) end
end})
SpR:Space({Size=8})
SpR:Button({Title="Detener", Icon="eye-off", Justify="Center", Callback=function() pcall(function() workspace.CurrentCamera.CameraSubject=Humanoid end); WindUI:Notify({Title="Espectear", Content="Detenido", Duration=2}) end})
Uti:Space({Size=8})
TPPlayerDropdown = Uti:Dropdown({Title="TP a Jugador", Values=GetPlayerNames(false), Value=1, Callback=function(s) TPPlayerName=s end})
Uti:Space({Size=6})
Uti:Button({Title="Teletransportar a Jugador", Icon="map-pin", Justify="Center", Callback=function()
    if not TPPlayerName or TPPlayerName=="No hay jugadores" then WindUI:Notify({Title="TP", Content="Selecciona un jugador", Duration=2}); return end
    local t=Players:FindFirstChild(TPPlayerName)
    if t and t.Character and t.Character:FindFirstChild("HumanoidRootPart") and RootPart then RootPart.CFrame=t.Character.HumanoidRootPart.CFrame*CFrame.new(0,0,4); WindUI:Notify({Title="TP", Content="A "..TPPlayerName, Duration=2})
    else WindUI:Notify({Title="Error", Content="Jugador no disponible", Duration=2}) end
end})
Uti:Space({Size=6})
Uti:Button({Title="Recargar Listas de Jugadores", Icon="refresh-cw", Justify="Center", Callback=RefreshTargetDropdowns})
Uti:Space({Size=8})
Uti:Button({Title="Server Hop", Icon="shuffle", Justify="Center", Callback=ServerHop}); Uti:Space({Size=6})
local UB1=Uti:Group({})
UB1:Button({Title="Copiar Link del Juego", Icon="link", Justify="Center", Callback=function() setclipboard("https://www.roblox.com/games/"..tostring(game.PlaceId)); WindUI:Notify({Title="Copiado", Content="Link copiado", Duration=2}) end})
UB1:Space({Size=8})
UB1:Button({Title="Copiar Link del Servidor", Icon="link-2", Justify="Center", Callback=function() setclipboard("roblox://placeId="..tostring(game.PlaceId).."&jobId="..tostring(game.JobId)); WindUI:Notify({Title="Copiado", Content="Link copiado", Duration=2}) end})
Uti:Space({Size=6})
local UB2=Uti:Group({})
UB2:Button({Title="Rejoin", Icon="refresh-cw", Justify="Center", Callback=function() TeleportService:TeleportToPlaceInstance(game.PlaceId, game.JobId) end})
UB2:Space({Size=8})
UB2:Button({Title="Volver al Spawn", Icon="home", Justify="Center", Callback=function() if RootPart then RootPart.CFrame=SpawnCFrame; RootPart.Velocity=Vector3.new(0,0,0); WindUI:Notify({Title="Spawn", Content="Volviste", Duration=2}) end end})
Uti:Space({Size=8})
CoordsParagraph = Uti:Paragraph({Title="Coordenadas Actuales", Desc="X: 0  Y: 0  Z: 0", Image="map", ImageSize=14})
H:Space({Size=10})
H:Section({Title="Movimiento Extra", TextSize=20}); H:Space({Size=6})
local ME=H:Section({Title="Movimiento Adicional", Box=true, BoxBorder=true, Opened=true})
ME:Toggle({Title="Click TP (Clic Derecho)", Def=Get("ClickTPEnabled", false), Callback=SetClickTP}); ME:Space({Size=6})
ME:Toggle({Title="Auto-Jump (Bunny Hop)", Def=Get("AutoJumpEnabled", false), Callback=function(s) AutoJumpEnabled=s end}); ME:Space({Size=6})
ME:Toggle({Title="Walk on Water", Def=Get("WalkOnWaterEnabled", false), Callback=function(s) WalkOnWaterEnabled=s end}); ME:Space({Size=6})
ME:Toggle({Title="Spin Bot", Def=Get("SpinBotEnabled", false), Callback=SetSpinBot}); ME:Space({Size=6})
ME:Toggle({Title="Freeze Position", Def=Get("FreezePositionEnabled", false), Callback=SetFreeze}); ME:Space({Size=6})
ME:Slider({Title="Velocidad de Caída Máx.", Step=10, Value={Min=10,Max=200,Default=Get("FallSpeedCap", 200)}, Callback=function(v) FallSpeedCap=v end}); ME:Space({Size=6})
ME:Slider({Title="Tamaño de Personaje", Step=0.1, Value={Min=0.3,Max=5,Default=Get("BodyScale", 1)}, Callback=function(v) pcall(function() if Humanoid then Humanoid.BodyHeightScale=v; Humanoid.BodyWidthScale=v; Humanoid.BodyDepthScale=v; if Humanoid.HeadScale then Humanoid.HeadScale=v end end end) end})
H:Space({Size=10})
H:Section({Title="Visual Extra", TextSize=20}); H:Space({Size=6})
local VE=H:Section({Title="Efectos Adicionales", Box=true, BoxBorder=true, Opened=true})
VE:Slider({Title="Velocidad de Animación", Step=0.1, Value={Min=0.1,Max=5,Default=Get("AnimationSpeed", 1)}, Callback=function(v) if Humanoid then pcall(function() Humanoid.AnimationSpeed=v end) end end}); VE:Space({Size=6})
VE:Slider({Title="Hora del Día (ClockTime)", Step=1, Value={Min=0,Max=24,Default=Get("ClockTime", 14)}, Callback=function(v) Lighting.ClockTime=v end}); VE:Space({Size=6})
VE:Button({Title="Eliminar Partículas/Efectos", Icon="trash-2", Justify="Center", Callback=RemoveParticles})
H:Space({Size=10})
H:Section({Title="Utilidades Extra", TextSize=20}); H:Space({Size=6})
local UE=H:Section({Title="Herramientas Adicionales", Box=true, BoxBorder=true, Opened=true})
UE:Toggle({Title="God Mode (Local)", Def=Get("GodModeEnabled", false), Callback=function(s) GodModeEnabled=s end}); UE:Space({Size=6})
UE:Toggle({Title="Auto-Respawn Instantáneo", Def=Get("InstantRespawnEnabled", false), Callback=function(s) InstantRespawnEnabled=s end}); UE:Space({Size=6})
UE:Toggle({Title="Seguir Jugador (Follow)", Def=Get("FollowPlayerEnabled", false), Callback=function(s) FollowPlayerEnabled=s end}); UE:Space({Size=8})
pcall(function() UE:TextBox({Title="Coordenadas (X, Y, Z)", PlaceholderText="0, 10, 0", Callback=function(t) CoordsText=t end}) end)
UE:Space({Size=6})
UE:Button({Title="TP a Coordenadas", Icon="map-pin", Justify="Center", Callback=function() TeleportToCoords(CoordsText) end}); UE:Space({Size=6})
local SvR=UE:Group({})
SvR:Button({Title="Guardar Posición", Icon="save", Justify="Center", Callback=function() if RootPart then SavedPosition=RootPart.CFrame end; WindUI:Notify({Title="Guardado", Content="Posición guardada", Duration=2}) end})
SvR:Space({Size=8})
SvR:Button({Title="Volver a Posición", Icon="home", Justify="Center", Callback=function() if SavedPosition and RootPart then RootPart.CFrame=SavedPosition; RootPart.Velocity=Vector3.new(0,0,0); WindUI:Notify({Title="TP", Content="Volviste", Duration=2}) else WindUI:Notify({Title="Error", Content="No hay posición guardada", Duration=2}) end end})
H:Space({Size=12})

-- 10. TROLLEOS
local T = Window:Tab({Title="Trolleos", Icon="laugh"})
T:Section({Title="Diversión y Trolleos", TextSize=20}); T:Space({Size=6})
local TC=T:Section({Title="Chat y Mensajes", Box=true, BoxBorder=true, Opened=true})
pcall(function() TC:TextBox({Title="Mensaje de Spam", PlaceholderText="Escribe tu mensaje...", Callback=function(t) SpamText=t end}) end)
TC:Space({Size=6})
TC:Toggle({Title="Chat Spammer", Def=Get("ChatSpamEnabled", false), Callback=SetChatSpam}); TC:Space({Size=6})
TC:Toggle({Title="Dance/Emote Spam", Def=Get("DanceSpamEnabled", false), Callback=SetDanceSpam}); TC:Space({Size=6})
TC:Toggle({Title="Chat Echo (Repetir)", Def=Get("ChatEchoEnabled", false), Callback=SetChatEcho})
T:Space({Size=10})
local TV=T:Section({Title="Visuales Graciosos", Box=true, BoxBorder=true, Opened=true})
TV:Toggle({Title="Personaje Arcoíris", Def=Get("RainbowEnabled", false), Callback=SetRainbow}); TV:Space({Size=6})
TV:Toggle({Title="Cabeza Grande", Def=Get("BigHeadEnabled", false), Callback=SetBigHead}); TV:Space({Size=6})
TV:Toggle({Title="Cámara Borracha", Def=Get("DrunkCamEnabled", false), Callback=SetDrunkCam}); TV:Space({Size=6})
TV:Toggle({Title="Temblor de Pantalla", Def=Get("ScreenShakeEnabled", false), Callback=SetScreenShake}); TV:Space({Size=6})
TV:Toggle({Title="Hacerse el Muerto", Def=Get("PlayDeadEnabled", false), Callback=SetPlayDead})
T:Space({Size=10})
local TU=T:Section({Title="Otros Trolleos", Box=true, BoxBorder=true, Opened=true})
MorphDropdown = TU:Dropdown({Title="Jugador para Morph", Values=GetPlayerNames(false), Value=1, Callback=function(s) MorphName=s end})
TU:Space({Size=6})
TU:Button({Title="Morfearse como Jugador", Icon="user", Justify="Center", Callback=MorphAsPlayer}); TU:Space({Size=6})
TU:Button({Title="Fake Kick (Pantalla Falsa)", Icon="alert-triangle", Justify="Center", Callback=FakeKick}); TU:Space({Size=6})
TU:Button({Title="Recargar Lista Jugadores", Icon="refresh-cw", Justify="Center", Callback=RefreshTargetDropdowns})
T:Space({Size=12})

-- 11. CRÉDITOS
local Cr = Window:Tab({Title="Créditos", Icon="award"})
Cr:Section({Title="Agradecimientos", TextSize=20}); Cr:Space({Size=6})
local CG=Cr:Group({})
CG:Paragraph({Title="Creador", Desc="ALAN_FF168\n© 2026", Image="code", ImageSize=16}); CG:Space({Size=10})
CG:Paragraph({Title="UI Library", Desc="WindUI v1.6.65\nFootagesus", Image="book", ImageSize=16})
Cr:Space({Size=8})
Cr:Paragraph({Title="Gracias por usar", Desc="¡Disfruta el script!", Image="heart", ImageSize=16})
Cr:Space({Size=12})
Cr:Paragraph({Title="", Desc="tonto el que ha leído esto", Image="smile", ImageSize=16})

-- Loop en tiempo real + auto-guardado + info del target
local SaveCounter = 0
task.spawn(function()
    while task.wait(0.1) do
        SaveCounter = SaveCounter + 0.1
        if SaveCounter >= 10 then
            SaveCounter = 0
            task.spawn(function() GuardarConfiguracion(true) end)
        end
        if not Character or not RootPart then UpdateChar() end
        if NoFrictionEnabled and RootPart then RootPart.Friction=0; RootPart.AirFriction=0
        elseif RootPart then RootPart.Friction=1; RootPart.AirFriction=0.5 end
        if (NoPushEnabled or NoKnockbackEnabled) and RootPart then RootPart.CustomPhysicalProperties=PhysicalProperties.new(10, 0.3, NoPushEnabled and 0 or 0.5)
        elseif RootPart then RootPart.CustomPhysicalProperties=PhysicalProperties.new(1, 0.5, 0.5) end
        if AntiVoidEnabled and RootPart then
            if RootPart.Position.Y>-10 then LastSafePos=RootPart.Position
            elseif RootPart.Position.Y<-50 then RootPart.CFrame=CFrame.new(LastSafePos+Vector3.new(0,5,0)); RootPart.Velocity=Vector3.new(0,0,0) end
        end
        if AntiRagdollEnabled and Humanoid then
            local st=Humanoid:GetState()
            if st==Enum.HumanoidStateType.Ragdoll or st==Enum.HumanoidStateType.FallingDown or st==Enum.HumanoidStateType.Physics then Humanoid:ChangeState(Enum.HumanoidStateType.GettingUp) end
        end
        if GodModeEnabled and Humanoid then pcall(function() Humanoid.Health=Humanoid.MaxHealth end) end
        if AutoJumpEnabled and Humanoid then pcall(function() if Humanoid.FloorMaterial~=Enum.Material.Air then Humanoid.Jump=true end end) end
        if WalkOnWaterEnabled and Humanoid and RootPart then pcall(function()
            if Humanoid:GetState()==Enum.HumanoidStateType.Swimming then Humanoid:ChangeState(Enum.HumanoidStateType.Running); RootPart.Velocity=Vector3.new(RootPart.Velocity.X, 50, RootPart.Velocity.Z) end
        end) end
        if RootPart and RootPart.Velocity.Y<-FallSpeedCap then RootPart.Velocity=Vector3.new(RootPart.Velocity.X, -FallSpeedCap, RootPart.Velocity.Z) end
        if FollowPlayerEnabled and RootPart then
            local t=TPPlayerName and Players:FindFirstChild(TPPlayerName)
            if t and t.Character and t.Character:FindFirstChild("HumanoidRootPart") then RootPart.CFrame=t.Character.HumanoidRootPart.CFrame*CFrame.new(0,0,5) end
        end
        if CoordsParagraph and RootPart and CoordsParagraph.SetDesc then
            local p=RootPart.Position; CoordsParagraph:SetDesc("X: "..math.floor(p.X).."  Y: "..math.floor(p.Y).."  Z: "..math.floor(p.Z))
        end
        if TargetInfoParagraph and TargetInfoParagraph.SetDesc then
            local txt = "Ninguno"
            local t = TargetName and Players:FindFirstChild(TargetName)
            if t and t.Character then
                local hrp = t.Character:FindFirstChild("HumanoidRootPart")
                local hum = t.Character:FindFirstChildOfClass("Humanoid")
                local dist = (hrp and RootPart) and math.floor((RootPart.Position-hrp.Position).Magnitude) or 0
                local hp = hum and (math.floor(hum.Health).."/"..math.floor(hum.MaxHealth)) or "?"
                txt = t.Name.."\nDistancia: "..dist.." m\nVida: "..hp
            end
            TargetInfoParagraph:SetDesc(txt)
        end
    end
end)

local function HookDeath(char)
    local hum=char:WaitForChild("Humanoid", 10)
    if hum then hum.Died:Connect(function()
        if InstantRespawnEnabled then task.wait(0.1); pcall(function() LocalPlayer:LoadCharacter() end); return end
        if AutoRejoinEnabled then task.wait(3); pcall(function() TeleportService:TeleportToPlaceInstance(game.PlaceId, game.JobId) end) end
    end) end
end
if Character then HookDeath(Character) end
LocalPlayer.CharacterAdded:Connect(HookDeath)

pcall(function()
    local SideBar=Window.SideBar
    if SideBar and SideBar.Container then
        local layout=SideBar.Container:FindFirstChildOfClass("UIListLayout")
        if layout then layout.Padding=UDim.new(0,12); layout.Spacing=UDim.new(0,18) end
    end
end)

AplicarConfiguracion()

WindUI:Notify({Title="DENJI•ALEX", Content="v11: Pestaña Terget + Funciones en Game", Duration=4})

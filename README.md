--[[
  RaioVerse Hub - Versão atualizada
  Removidas: Correr mais rápido, Pulo infinito
  Adicionada: Imortalidade (Vida infinita) - tenta manter sua vida sempre cheia
  Mantidas: AimBot (visível apenas), ESP Esqueleto, Minimizar, FOV, NoClip, ClickTP, Anti-AFK
  Coloque esse script como LocalScript em StarterGui.
--]]

local Players        = game:GetService("Players")
local RunService     = game:GetService("RunService")
local Workspace      = game:GetService("Workspace")
local CoreGui        = game:GetService("CoreGui")
local UserInput      = game:GetService("UserInputService")
local VirtualUser    = game:GetService("VirtualUser")

local localPlayer = Players.LocalPlayer
local camera = Workspace.CurrentCamera

-- ===== Configs =====
local AIM_SMOOTH = 0.12            -- suavidade do aimbot (0.01 lento .. 1 instantâneo)
local AIM_FOV_PIXELS = 400         -- somente alvos dentro desse raio (px) do centro
local SKELETON_WIDTH0 = 0.15
local SKELETON_WIDTH1 = 0.06
local SKELETON_ALPHA = 0.22
local IMMORTAL_MAX_HEALTH = 1e6    -- valor atribuído ao MaxHealth local (ajuste se quiser)
local IMMORTAL_RESTORE_DELAY = 0.03 -- delay entre tentativas de restauração de vida

-- ===== Paleta e estilos =====
local COLORS = {
    bg = Color3.fromRGB(18,20,30),
    header = Color3.fromRGB(28,32,52),
    accent = Color3.fromRGB(115,150,255),
    btn = Color3.fromRGB(60,72,150),
    btn_hover = Color3.fromRGB(85,105,220),
    text = Color3.fromRGB(232,240,255),
    soft = Color3.fromRGB(40,44,60),
}

-- ===== GUI =====
local gui = Instance.new("ScreenGui")
gui.Name = "RaioVerseHub"
gui.IgnoreGuiInset = true
gui.Parent = CoreGui
gui.DisplayOrder = 9999

-- Shadow
local shadow = Instance.new("Frame", gui)
shadow.Size = UDim2.new(0, 420, 0, 260)
shadow.Position = UDim2.new(0.5, -210, 0.5, -130)
shadow.BackgroundColor3 = Color3.fromRGB(6,6,10)
shadow.BackgroundTransparency = 0.6
shadow.ZIndex = 0
local sc = Instance.new("UICorner", shadow); sc.CornerRadius = UDim.new(0, 18)

-- Main
local main = Instance.new("Frame", gui)
main.Size = UDim2.new(0, 420, 0, 260)
main.Position = UDim2.new(0.5, -210, 0.5, -130)
main.BackgroundColor3 = COLORS.bg
main.BorderSizePixel = 0
main.ZIndex = 1
local mc = Instance.new("UICorner", main); mc.CornerRadius = UDim.new(0, 18)
main.Active = true
main.Draggable = true

-- Header
local header = Instance.new("Frame", main)
header.Size = UDim2.new(1, 0, 0, 48)
header.Position = UDim2.new(0, 0, 0, 0)
header.BackgroundColor3 = COLORS.header
header.BorderSizePixel = 0
local hc = Instance.new("UICorner", header); hc.CornerRadius = UDim.new(0, 14)

local title = Instance.new("TextLabel", header)
title.Size = UDim2.new(1, -120, 1, 0)
title.Position = UDim2.new(0, 18, 0, 0)
title.BackgroundTransparency = 1
title.Font = Enum.Font.GothamBold
title.Text = "RaioVerse — Clean"
title.TextSize = 20
title.TextColor3 = COLORS.accent
title.TextXAlignment = Enum.TextXAlignment.Left

local sub = Instance.new("TextLabel", header)
sub.Size = UDim2.new(0, 100, 1, 0)
sub.Position = UDim2.new(1, -120, 0, 0)
sub.BackgroundTransparency = 1
sub.Font = Enum.Font.Gotham
sub.Text = "v1.1"
sub.TextSize = 14
sub.TextColor3 = COLORS.text
sub.TextXAlignment = Enum.TextXAlignment.Right

-- Minimize
local minBtn = Instance.new("TextButton", header)
minBtn.Size = UDim2.new(0, 38, 0, 30)
minBtn.Position = UDim2.new(1, -52, 0, 9)
minBtn.BackgroundColor3 = COLORS.btn
minBtn.Text = "—"
minBtn.Font = Enum.Font.GothamBold
minBtn.TextColor3 = COLORS.text
minBtn.TextSize = 20
local mbc = Instance.new("UICorner", minBtn); mbc.CornerRadius = UDim.new(0, 8)

-- Close
local closeBtn = Instance.new("TextButton", header)
closeBtn.Size = UDim2.new(0, 28, 0, 28)
closeBtn.Position = UDim2.new(1, -12, 0, 10)
closeBtn.BackgroundColor3 = Color3.fromRGB(180,60,70)
closeBtn.Text = "✕"
closeBtn.Font = Enum.Font.SourceSansBold
closeBtn.TextSize = 14
closeBtn.TextColor3 = Color3.fromRGB(255,255,255)
Instance.new("UICorner", closeBtn).CornerRadius = UDim.new(0, 8)

-- Container
local container = Instance.new("Frame", main)
container.Position = UDim2.new(0, 18, 0, 60)
container.Size = UDim2.new(1, -36, 1, -78)
container.BackgroundTransparency = 1

local layout = Instance.new("UIListLayout", container)
layout.Padding = UDim.new(0, 10)
layout.SortOrder = Enum.SortOrder.LayoutOrder

local function makeToggleBtn(text)
    local btn = Instance.new("TextButton", container)
    btn.Size = UDim2.new(1, 0, 0, 40)
    btn.BackgroundColor3 = COLORS.btn
    btn.AutoButtonColor = true
    btn.Font = Enum.Font.GothamSemibold
    btn.Text = "[ OFF ]  " .. text
    btn.TextColor3 = COLORS.text
    btn.TextSize = 15
    local c = Instance.new("UICorner", btn); c.CornerRadius = UDim.new(0, 10)
    btn.MouseEnter:Connect(function() btn.BackgroundColor3 = COLORS.btn_hover end)
    btn.MouseLeave:Connect(function() btn.BackgroundColor3 = COLORS.btn end)
    return btn
end

-- Buttons
local aimBtn = makeToggleBtn("AimBot Suave (visível apenas)")
local skeletonBtn = makeToggleBtn("ESP Esqueleto (Hitbox)")
local immortalBtn = makeToggleBtn("Imortal (Vida infinita)")

-- Extra 4 funções universais restantes
local fovBtn        = makeToggleBtn("FOV + (toggle)")
local noclipBtn     = makeToggleBtn("NoClip (toggle)")
local clickTPBtn    = makeToggleBtn("ClickTP (teleport ao clicar)")
local antiAfkBtn    = makeToggleBtn("Anti-AFK (toggle)")

-- Minimized icon
local icon = Instance.new("ImageButton", gui)
icon.Size = UDim2.new(0, 28, 0, 28)
icon.Position = UDim2.new(0, 14, 0, 14)
icon.BackgroundTransparency = 0.55
icon.Image = "rbxassetid://3926305904"
icon.Visible = false
icon.ZIndex = 50
Instance.new("UICorner", icon).CornerRadius = UDim.new(0, 8)

-- Credits
local credit = Instance.new("TextLabel", main)
credit.Size = UDim2.new(1, -28, 0, 18)
credit.Position = UDim2.new(0, 16, 1, -26)
credit.BackgroundTransparency = 1
credit.Font = Enum.Font.Gotham
credit.Text = "☄️ RaioVerse"
credit.TextSize = 12
credit.TextColor3 = COLORS.accent
credit.TextXAlignment = Enum.TextXAlignment.Left

-- ===== State =====
local State = {
    Aim = false,
    Skeleton = false,
    Immortal = false,
    FOV = false,
    NoClip = false,
    ClickTP = false,
    AntiAFK = false,
}

-- ===== Minimizar logic =====
minBtn.MouseButton1Click:Connect(function()
    main.Visible = false
    shadow.Visible = false
    icon.Visible = true
end)
icon.MouseButton1Click:Connect(function()
    main.Visible = true
    shadow.Visible = true
    icon.Visible = false
end)
closeBtn.MouseButton1Click:Connect(function()
    pcall(function() gui:Destroy() end)
end)

-- ===== Aimbot (aponta somente para players visíveis) =====
local aimConnection = nil
local function isPlayerVisible(target)
    if not target or not target.Character then return false end
    local hrp = target.Character:FindFirstChild("HumanoidRootPart")
    local humanoid = target.Character:FindFirstChildOfClass("Humanoid")
    if not hrp or not humanoid or humanoid.Health <= 0 then return false end

    -- on screen
    local screenPos, onScreen = camera:WorldToViewportPoint(hrp.Position)
    if not onScreen then return false end

    -- FOV check (pixels from center)
    local center = Vector2.new(camera.ViewportSize.X/2, camera.ViewportSize.Y/2)
    local dist = (Vector2.new(screenPos.X, screenPos.Y) - center).Magnitude
    if dist > AIM_FOV_PIXELS then return false end

    -- Raycast: camera -> target to ensure no walls
    local origin = camera.CFrame.Position
    local direction = (hrp.Position - origin)
    local params = RaycastParams.new()
    params.FilterType = Enum.RaycastFilterType.Blacklist
    params.FilterDescendantsInstances = { localPlayer.Character }
    params.IgnoreWater = true
    local res = Workspace:Raycast(origin, direction, params)
    if res then
        if res.Instance and res.Instance:IsDescendantOf(target.Character) then
            return true
        else
            return false
        end
    end
    return true
end

local function getClosestVisiblePlayer()
    local best = nil
    local bestDist = math.huge
    local center = Vector2.new(camera.ViewportSize.X/2, camera.ViewportSize.Y/2)
    for _, p in ipairs(Players:GetPlayers()) do
        if p ~= localPlayer and p.Character and p.Character:FindFirstChild("HumanoidRootPart") then
            if isPlayerVisible(p) then
                local sp = camera:WorldToViewportPoint(p.Character.HumanoidRootPart.Position)
                local d = (Vector2.new(sp.X, sp.Y) - center).Magnitude
                if d < bestDist then
                    best = p
                    bestDist = d
                end
            end
        end
    end
    return best
end

local function aimStep()
    if not State.Aim then return end
    if not camera then camera = Workspace.CurrentCamera end
    local target = getClosestVisiblePlayer()
    if target and target.Character and target.Character:FindFirstChild("HumanoidRootPart") then
        local hrp = target.Character.HumanoidRootPart
        local camPos = camera.CFrame.Position
        local desired = CFrame.new(camPos, hrp.Position)
        camera.CFrame = camera.CFrame:Lerp(desired, AIM_SMOOTH)
    end
end

-- ===== Skeleton ESP =====
local skeletons = {} -- skeletons[player] = {attachments = {...}, beams = {...}}
local bones = {
    {"Head","UpperTorso"}, {"UpperTorso","LowerTorso"},
    {"UpperTorso","LeftUpperArm"}, {"LeftUpperArm","LeftLowerArm"}, {"LeftLowerArm","LeftHand"},
    {"UpperTorso","RightUpperArm"}, {"RightUpperArm","RightLowerArm"}, {"RightLowerArm","RightHand"},
    {"LowerTorso","LeftUpperLeg"}, {"LeftUpperLeg","LeftLowerLeg"}, {"LeftLowerLeg","LeftFoot"},
    {"LowerTorso","RightUpperLeg"}, {"RightUpperLeg","RightLowerLeg"}, {"RightLowerLeg","RightFoot"},
}

local function createSkeletonFor(player)
    if not player or player == localPlayer then return end
    if skeletons[player] then return end
    local char = player.Character
    if not char then return end
    local data = { attachments = {}, beams = {} }

    for _, pair in ipairs(bones) do
        local aName, bName = pair[1], pair[2]
        local partA = char:FindFirstChild(aName)
        local partB = char:FindFirstChild(bName)
        if partA and partB then
            local attA = Instance.new("Attachment")
            attA.Name = "Raio_Att_"..aName
            attA.Parent = partA
            local attB = Instance.new("Attachment")
            attB.Name = "Raio_Att_"..bName
            attB.Parent = partB

            local beam = Instance.new("Beam")
            beam.Name = "Raio_Beam_"..aName.."_"..bName
            beam.Attachment0 = attA
            beam.Attachment1 = attB
            beam.FaceCamera = true
            beam.Width0 = SKELETON_WIDTH0
            beam.Width1 = SKELETON_WIDTH1
            beam.Transparency = NumberSequence.new(1 - (1 - SKELETON_ALPHA))
            beam.Color = ColorSequence.new(COLORS.accent)
            beam.LightEmission = 0.35
            beam.ZIndex = 3
            beam.Parent = Workspace

            table.insert(data.attachments, attA)
            table.insert(data.attachments, attB)
            table.insert(data.beams, beam)
        end
    end

    skeletons[player] = data
end

local function destroySkeletonFor(player)
    local data = skeletons[player]
    if not data then return end
    for _, att in ipairs(data.attachments) do
        pcall(function() att:Destroy() end)
    end
    for _, beam in ipairs(data.beams) do
        pcall(function() beam:Destroy() end)
    end
    skeletons[player] = nil
end

local function enableSkeletons()
    for _, p in ipairs(Players:GetPlayers()) do
        if p ~= localPlayer and p.Character then
            createSkeletonFor(p)
        end
    end
end

local function disableSkeletons()
    for p, _ in pairs(skeletons) do
        destroySkeletonFor(p)
    end
end

-- sync players
Players.PlayerAdded:Connect(function(p)
    p.CharacterAdded:Connect(function()
        if State.Skeleton then
            wait(0.12)
            createSkeletonFor(p)
        end
    end)
end)
Players.PlayerRemoving:Connect(function(p) destroySkeletonFor(p) end)
for _, p in ipairs(Players:GetPlayers()) do
    if p ~= localPlayer then
        p.CharacterAdded:Connect(function() if State.Skeleton then wait(0.12); createSkeletonFor(p) end end)
    end
end

-- ===== Immortality (client-side attempt) =====
local immortalConn = nil
local humanoidHealthConns = {}

local function enableImmortal()
    -- hook player's humanoid
    local function hookHumanoid(hum)
        if not hum then return end
        pcall(function() hum.MaxHealth = IMMORTAL_MAX_HEALTH end)
        pcall(function() hum.Health = hum.MaxHealth end)
        -- HealthChanged: restore if drops
        if humanoidHealthConns[hum] then humanoidHealthConns[hum]:Disconnect() end
        humanoidHealthConns[hum] = hum.HealthChanged:Connect(function(h)
            -- tiny wait to avoid race
            wait(IMMORTAL_RESTORE_DELAY)
            if hum and hum.Parent then
                pcall(function()
                    if hum.Health < hum.MaxHealth then
                        hum.Health = hum.MaxHealth
                    end
                end)
            end
        end)
        -- also prevent death by setting Died handler to immediately restore (best-effort)
        hum.Died:Connect(function()
            wait(0.03)
            if hum and hum.Parent then
                pcall(function()
                    hum.Health = hum.MaxHealth
                end)
            end
        end)
    end

    if localPlayer.Character then
        local hum = localPlayer.Character:FindFirstChildOfClass("Humanoid")
        hookHumanoid(hum)
    end
    -- reconnect on respawn
    localPlayer.CharacterAdded:Connect(function(char)
        wait(0.2)
        local hum = char:FindFirstChildOfClass("Humanoid")
        hookHumanoid(hum)
    end)
    -- also attempt to hook existing humanoid now (if any)
    if localPlayer.Character and localPlayer.Character:FindFirstChildOfClass("Humanoid") then
        hookHumanoid(localPlayer.Character:FindFirstChildOfClass("Humanoid"))
    end

    -- global guard: ensure periodic restore (best-effort client-side)
    if immortalConn then immortalConn:Disconnect() end
    immortalConn = RunService.Heartbeat:Connect(function()
        local char = localPlayer.Character
        if char then
            local hum = char:FindFirstChildOfClass("Humanoid")
            if hum and hum.Health < hum.MaxHealth then
                pcall(function() hum.Health = hum.MaxHealth end)
            end
            if hum and hum.MaxHealth < IMMORTAL_MAX_HEALTH then
                pcall(function() hum.MaxHealth = IMMORTAL_MAX_HEALTH end)
            end
        end
    end)
end

local function disableImmortal()
    if immortalConn then immortalConn:Disconnect(); immortalConn = nil end
    for hum, conn in pairs(humanoidHealthConns) do
        pcall(function() conn:Disconnect() end)
    end
    humanoidHealthConns = {}
    -- try to reset MaxHealth to default
    if localPlayer.Character then
        local hum = localPlayer.Character:FindFirstChildOfClass("Humanoid")
        if hum then
            pcall(function() hum.MaxHealth = 100; if hum.Health > hum.MaxHealth then hum.Health = hum.MaxHealth end end)
        end
    end
end

-- ===== Universal Features =====

-- 1) FOV changer (toggle)
local DEFAULT_FOV = 70
local FOV_TARGET = 110
local function setFOV(on)
    if not camera then camera = Workspace.CurrentCamera end
    if on then camera.FieldOfView = FOV_TARGET else camera.FieldOfView = DEFAULT_FOV end
end

-- 2) NoClip (toggle) - set CanCollide false client-side
local noclipConn = nil
local function applyNoClip()
    local char = localPlayer.Character
    if not char then return end
    for _, part in ipairs(char:GetDescendants()) do
        if part:IsA("BasePart") then
            pcall(function() part.CanCollide = false end)
        end
    end
end
local function startNoClip()
    if noclipConn then return end
    applyNoClip()
    noclipConn = RunService.Stepped:Connect(function() applyNoClip() end)
end
local function stopNoClip()
    if noclipConn then noclipConn:Disconnect(); noclipConn = nil end
end

-- 3) ClickTP (teleport to mouse position when clicking)
local clickTP_Conn = nil
local mouse = localPlayer:GetMouse()
local function startClickTP()
    if clickTP_Conn then return end
    clickTP_Conn = mouse.Button1Down:Connect(function()
        if not State.ClickTP then return end
        local pos = mouse.Hit and mouse.Hit.Position
        if pos and localPlayer.Character and localPlayer.Character:FindFirstChild("HumanoidRootPart") then
            local hrp = localPlayer.Character.HumanoidRootPart
            hrp.CFrame = CFrame.new(pos + Vector3.new(0, 3, 0))
        end
    end)
end
local function stopClickTP()
    if clickTP_Conn then clickTP_Conn:Disconnect(); clickTP_Conn = nil end
end

-- 4) Anti-AFK
local antiAfkConn = nil
local function startAntiAfk()
    if antiAfkConn then return end
    pcall(function() VirtualUser:CaptureController() end)
    antiAfkConn = localPlayer.Idled:Connect(function()
        pcall(function()
            VirtualUser:Button2Down(Vector2.new(0,0))
            wait(0.1)
            VirtualUser:Button2Up(Vector2.new(0,0))
        end)
    end)
end
local function stopAntiAfk()
    if antiAfkConn then antiAfkConn:Disconnect(); antiAfkConn = nil end
end

-- ===== Button events hookup =====
aimBtn.MouseButton1Click:Connect(function()
    State.Aim = not State.Aim
    aimBtn.Text = (State.Aim and "[ ON ]  AimBot Suave (visível)" or "[ OFF ]  AimBot Suave (visível)")
    if State.Aim then
        if not aimConnection then aimConnection = RunService.RenderStepped:Connect(aimStep) end
    else
        if aimConnection then aimConnection:Disconnect(); aimConnection = nil end
    end
end)

skeletonBtn.MouseButton1Click:Connect(function()
    State.Skeleton = not State.Skeleton
    skeletonBtn.Text = (State.Skeleton and "[ ON ]  ESP Esqueleto" or "[ OFF ]  ESP Esqueleto")
    if State.Skeleton then
        enableSkeletons()
    else
        disableSkeletons()
    end
end)

immortalBtn.MouseButton1Click:Connect(function()
    State.Immortal = not State.Immortal
    immortalBtn.Text = (State.Immortal and "[ ON ]  Imortal (Vida infinita)" or "[ OFF ]  Imortal (Vida infinita)")
    if State.Immortal then
        enableImmortal()
    else
        disableImmortal()
    end
end)

fovBtn.MouseButton1Click:Connect(function()
    State.FOV = not State.FOV
    fovBtn.Text = (State.FOV and "[ ON ]  FOV +" or "[ OFF ]  FOV +")
    setFOV(State.FOV)
end)

noclipBtn.MouseButton1Click:Connect(function()
    State.NoClip = not State.NoClip
    noclipBtn.Text = (State.NoClip and "[ ON ]  NoClip" or "[ OFF ]  NoClip")
    if State.NoClip then startNoClip() else stopNoClip() end
end)

clickTPBtn.MouseButton1Click:Connect(function()
    State.ClickTP = not State.ClickTP
    clickTPBtn.Text = (State.ClickTP and "[ ON ]  ClickTP" or "[ OFF ]  ClickTP")
    if State.ClickTP then startClickTP() else stopClickTP() end
end)

antiAfkBtn.MouseButton1Click:Connect(function()
    State.AntiAFK = not State.AntiAFK
    antiAfkBtn.Text = (State.AntiAFK and "[ ON ]  Anti-AFK" or "[ OFF ]  Anti-AFK")
    if State.AntiAFK then startAntiAfk() else stopAntiAfk() end
end)

-- Ensure features reapply on respawn
localPlayer.CharacterAdded:Connect(function(char)
    wait(0.25)
    camera = Workspace.CurrentCamera
    if State.NoClip then startNoClip() end
    if State.Skeleton then enableSkeletons() end
    if State.Immortal then enableImmortal() end
end)

-- Cleanup on GUI removal
gui.Destroying:Connect(function()
    if aimConnection then aimConnection:Disconnect(); aimConnection = nil end
    disableSkeletons()
    stopNoClip()
    stopClickTP()
    stopAntiAfk()
    disableImmortal()
    setFOV(false) -- reset FOV
end)

print("[RaioVerseHub] carregado — Imortalidade: melhor esforço client-side. Use com responsabilidade.")

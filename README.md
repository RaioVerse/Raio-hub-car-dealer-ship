--[[
  RaioVerse Hub - Leve, bonito e funcional
  Funções:
    - AimBot suave (só mira players visíveis, sem atravessar paredes)
    - Speed (WalkSpeed toggle)
    - Skeleton ESP (esqueleto) em tempo real
    - Minimizar eficiente (mostra ícone discreto)
  Instruções: cole este script como LocalScript em StarterGui
--]]

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local CoreGui = game:GetService("CoreGui")
local Workspace = game:GetService("Workspace")

local localPlayer = Players.LocalPlayer
local camera = Workspace.CurrentCamera

-- === Configs ===
local AIM_SMOOTH = 0.12         -- 0.01 (muito lento) -> 1 (instantâneo)
local AIM_FOV_PIXELS = 400     -- só considera alvos dentro desse raio (pixels) do centro da tela
local SPEED_VALUE = 45

-- Paleta + estilos leves
local COLORS = {
    bg = Color3.fromRGB(24,26,38),
    header = Color3.fromRGB(34,38,62),
    accent = Color3.fromRGB(119,152,255),
    btn = Color3.fromRGB(70,80,145),
    btn_hover = Color3.fromRGB(95,120,220),
    text = Color3.fromRGB(235,240,255),
}

-- === GUI ===
local gui = Instance.new("ScreenGui")
gui.Name = "RaioVerseHub"
gui.Parent = CoreGui
gui.DisplayOrder = 9999

-- shadow
local shadow = Instance.new("Frame", gui)
shadow.Size = UDim2.new(0, 360, 0, 220)
shadow.Position = UDim2.new(0.5, -180, 0.5, -110)
shadow.BackgroundColor3 = Color3.fromRGB(8,8,12)
shadow.BackgroundTransparency = 0.6
shadow.ZIndex = 0
local sc = Instance.new("UICorner", shadow); sc.CornerRadius = UDim.new(0, 16)

-- main
local main = Instance.new("Frame", gui)
main.Size = UDim2.new(0, 360, 0, 220)
main.Position = UDim2.new(0.5, -180, 0.5, -110)
main.BackgroundColor3 = COLORS.bg
main.BorderSizePixel = 0
main.ZIndex = 1
local mc = Instance.new("UICorner", main); mc.CornerRadius = UDim.new(0, 16)
main.Active = true
main.Draggable = true

-- header
local header = Instance.new("Frame", main)
header.Size = UDim2.new(1, 0, 0, 44)
header.Position = UDim2.new(0, 0, 0, 0)
header.BackgroundColor3 = COLORS.header
header.BorderSizePixel = 0
local hc = Instance.new("UICorner", header); hc.CornerRadius = UDim.new(0, 12)

local title = Instance.new("TextLabel", header)
title.Size = UDim2.new(1, -80, 1, 0)
title.Position = UDim2.new(0, 16, 0, 0)
title.BackgroundTransparency = 1
title.Font = Enum.Font.GothamBold
title.Text = "RaioVerse — Leve"
title.TextSize = 20
title.TextColor3 = COLORS.accent
title.TextXAlignment = Enum.TextXAlignment.Left

-- minimize button
local minBtn = Instance.new("TextButton", header)
minBtn.Size = UDim2.new(0, 36, 0, 28)
minBtn.Position = UDim2.new(1, -48, 0, 8)
minBtn.BackgroundColor3 = COLORS.btn
minBtn.Text = "—"
minBtn.Font = Enum.Font.GothamBold
minBtn.TextColor3 = COLORS.text
minBtn.TextSize = 20
local mbc = Instance.new("UICorner", minBtn); mbc.CornerRadius = UDim.new(0, 8)

-- container
local container = Instance.new("Frame", main)
container.Position = UDim2.new(0, 16, 0, 56)
container.Size = UDim2.new(1, -32, 1, -72)
container.BackgroundTransparency = 1

local list = Instance.new("UIListLayout", container)
list.Padding = UDim.new(0, 10)
list.SortOrder = Enum.SortOrder.LayoutOrder

local function makeButton(text, color)
    local b = Instance.new("TextButton", container)
    b.Size = UDim2.new(1, 0, 0, 40)
    b.BackgroundColor3 = color or COLORS.btn
    b.AutoButtonColor = true
    b.Font = Enum.Font.GothamSemibold
    b.Text = "[ OFF ]  " .. text
    b.TextColor3 = COLORS.text
    b.TextSize = 16
    local c = Instance.new("UICorner", b); c.CornerRadius = UDim.new(0, 8)
    b.MouseEnter:Connect(function() b.BackgroundColor3 = COLORS.btn_hover end)
    b.MouseLeave:Connect(function() b.BackgroundColor3 = color or COLORS.btn end)
    return b
end

-- minimize icon (small, non-intrusive)
local icon = Instance.new("ImageButton", gui)
icon.Size = UDim2.new(0, 28, 0, 28)
icon.Position = UDim2.new(0, 14, 0, 14)
icon.BackgroundTransparency = 0.55
icon.Image = "rbxassetid://3926305904"
icon.Visible = false
icon.ZIndex = 50

-- credits
local credit = Instance.new("TextLabel", main)
credit.Size = UDim2.new(1, -20, 0, 16)
credit.Position = UDim2.new(0, 10, 1, -22)
credit.BackgroundTransparency = 1
credit.Font = Enum.Font.Gotham
credit.Text = "☄️ RaioVerse"
credit.TextSize = 12
credit.TextColor3 = COLORS.accent
credit.TextXAlignment = Enum.TextXAlignment.Left

-- create buttons
local aimBtn = makeButton("AimBot Suave (visível somente)", COLORS.btn)
local speedBtn = makeButton("Correr mais rápido", COLORS.btn)
local skeletonBtn = makeButton("ESP Esqueleto (Hitbox)", COLORS.btn)

-- === State ===
local State = {
    Aim = false,
    Speed = false,
    Skeleton = false,
}

-- === Minimizar logic ===
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

-- === AIMBOT (visível somente) ===
local aimConnection = nil

local function isPlayerVisible(target)
    -- validações básicas
    if not target or not target.Character then return false end
    local hrp = target.Character:FindFirstChild("HumanoidRootPart")
    local humanoid = target.Character:FindFirstChildOfClass("Humanoid")
    if not hrp or not humanoid or humanoid.Health <= 0 then return false end

    -- está na tela?
    local screenPos, onScreen = camera:WorldToViewportPoint(hrp.Position)
    if not onScreen then return false end

    -- check FOV (distância em pixels do centro)
    local center = Vector2.new(camera.ViewportSize.X / 2, camera.ViewportSize.Y / 2)
    local dist = (Vector2.new(screenPos.X, screenPos.Y) - center).Magnitude
    if dist > AIM_FOV_PIXELS then return false end

    -- Raycast para verificar linha de visão (sem atravessar paredes)
    local origin = camera.CFrame.Position
    local direction = (hrp.Position - origin)
    local rayParams = RaycastParams.new()
    rayParams.FilterType = Enum.RaycastFilterType.Blacklist
    -- ignorar o próprio personagem
    if localPlayer.Character then
        rayParams.FilterDescendantsInstances = { localPlayer.Character }
    else
        rayParams.FilterDescendantsInstances = {}
    end
    rayParams.IgnoreWater = true

    local res = Workspace:Raycast(origin, direction, rayParams)
    if res then
        -- se acertou algo, aceitar somente se for parte do target.Character
        if res.Instance and res.Instance:IsDescendantOf(target.Character) then
            return true
        else
            return false
        end
    end
    -- sem hits (raro) -> considerar visível
    return true
end

local function getClosestVisiblePlayer()
    local best, bestDist = nil, math.huge
    local center = Vector2.new(camera.ViewportSize.X/2, camera.ViewportSize.Y/2)
    for _, p in ipairs(Players:GetPlayers()) do
        if p ~= localPlayer and p.Character and p.Character:FindFirstChild("HumanoidRootPart") then
            if isPlayerVisible(p) then
                local pos = camera:WorldToViewportPoint(p.Character.HumanoidRootPart.Position)
                local d = (Vector2.new(pos.X, pos.Y) - center).Magnitude
                if d < bestDist then
                    bestDist = d
                    best = p
                end
            end
        end
    end
    return best
end

local function aimStep(dt)
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

-- === SPEED ===
local function setSpeed(on)
    if localPlayer.Character then
        local humanoid = localPlayer.Character:FindFirstChildOfClass("Humanoid")
        if humanoid then
            humanoid.WalkSpeed = on and SPEED_VALUE or 16
        end
    end
end

-- === SKELETON ESP (efficient, create beams once per player) ===
local skeletons = {}
local bones = {
    {"Head","UpperTorso"}, {"UpperTorso","LowerTorso"},
    {"UpperTorso","LeftUpperArm"}, {"LeftUpperArm","LeftLowerArm"}, {"LeftLowerArm","LeftHand"},
    {"UpperTorso","RightUpperArm"}, {"RightUpperArm","RightLowerArm"}, {"RightLowerArm","RightHand"},
    {"LowerTorso","LeftUpperLeg"}, {"LeftUpperLeg","LeftLowerLeg"}, {"LeftLowerLeg","LeftFoot"},
    {"LowerTorso","RightUpperLeg"}, {"RightUpperLeg","RightLowerLeg"}, {"RightLowerLeg","RightFoot"},
}

local function createSkeletonFor(player)
    if skeletons[player] then return end
    if not player.Character then return end
    local data = { attachments = {}, beams = {} }

    -- create attachments per needed part (lazy: only if part exists)
    for _, bonePair in ipairs(bones) do
        local aName, bName = bonePair[1], bonePair[2]
        local partA = player.Character:FindFirstChild(aName)
        local partB = player.Character:FindFirstChild(bName)
        if partA and partB then
            local attA = Instance.new("Attachment")
            attA.Name = "RaioAtt_"..aName
            attA.Parent = partA
            local attB = Instance.new("Attachment")
            attB.Name = "RaioAtt_"..bName
            attB.Parent = partB

            local beam = Instance.new("Beam")
            beam.Name = "RaioBeam_"..aName.."_"..bName
            beam.Attachment0 = attA
            beam.Attachment1 = attB
            beam.FaceCamera = true
            beam.Width0 = 0.15
            beam.Width1 = 0.06
            beam.Transparency = NumberSequence.new(0.25)
            beam.Color = ColorSequence.new(COLORS.accent)
            beam.LightEmission = 0.4
            beam.Parent = Workspace -- parent in workspace so beams render well

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
    for _, b in ipairs(data.beams) do
        pcall(function() b:Destroy() end)
    end
    skeletons[player] = nil
end

local function enableAllSkeletons()
    -- create for existing players
    for _, p in ipairs(Players:GetPlayers()) do
        if p ~= localPlayer and p.Character then
            createSkeletonFor(p)
        end
    end
end

local function disableAllSkeletons()
    for p,_ in pairs(skeletons) do
        destroySkeletonFor(p)
    end
end

-- Keep skeletons in sync with character lifecycle
Players.PlayerAdded:Connect(function(p)
    p.CharacterAdded:Connect(function()
        if State.Skeleton then
            -- small delay to allow parts to exist
            wait(0.15)
            createSkeletonFor(p)
        end
    end)
end)
Players.PlayerRemoving:Connect(function(p)
    destroySkeletonFor(p)
end)

-- Also handle when other players respawn
local function onCharacterDescendantsChanged(player)
    -- if skeleton enabled and character changed, rebuild
    destroySkeletonFor(player)
    if State.Skeleton and player.Character then
        createSkeletonFor(player)
    end
end

-- monitor characters currently present
for _, p in ipairs(Players:GetPlayers()) do
    if p ~= localPlayer then
        p.CharacterAdded:Connect(function() if State.Skeleton then wait(0.12); createSkeletonFor(p) end end)
    end
end

-- === Button events ===
aimBtn.MouseButton1Click:Connect(function()
    State.Aim = not State.Aim
    aimBtn.Text = (State.Aim and "[ ON ]  AimBot Suave" or "[ OFF ]  AimBot Suave")
    if State.Aim and not aimConnection then
        aimConnection = RunService.RenderStepped:Connect(aimStep)
    elseif not State.Aim and aimConnection then
        aimConnection:Disconnect(); aimConnection = nil
    end
end)

speedBtn.MouseButton1Click:Connect(function()
    State.Speed = not State.Speed
    speedBtn.Text = (State.Speed and "[ ON ]  Correr mais rápido" or "[ OFF ]  Correr mais rápido")
    setSpeed(State.Speed)
end)

skeletonBtn.MouseButton1Click:Connect(function()
    State.Skeleton = not State.Skeleton
    skeletonBtn.Text = (State.Skeleton and "[ ON ]  ESP Esqueleto" or "[ OFF ]  ESP Esqueleto")
    if State.Skeleton then
        enableAllSkeletons()
        -- connect existing characters for rebuild on respawn
        for _, p in ipairs(Players:GetPlayers()) do
            if p ~= localPlayer then
                p.CharacterAdded:Connect(function()
                    -- small wait then ensure skeleton for this player
                    wait(0.12)
                    createSkeletonFor(p)
                end)
                -- also watch for character model change: rebuild skeleton when descendant structure changes
                if p.Character then
                    p.Character.DescendantAdded:Connect(function() onCharacterDescendantsChanged(p) end)
                    p.Character.DescendantRemoving:Connect(function() onCharacterDescendantsChanged(p) end)
                end
            end
        end
    else
        disableAllSkeletons()
    end
end)

-- Ensure WalkSpeed re-applied on respawn
localPlayer.CharacterAdded:Connect(function()
    wait(0.25)
    if State.Speed then setSpeed(true) end
end)

-- Cleanup on script disable/unload
gui.Destroying:Connect(function()
    if aimConnection then aimConnection:Disconnect() end
    disableAllSkeletons()
end)

-- end of script

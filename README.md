--[[
    RaioVerse Hub - Simples, bonito e leve!
    Funções: AimBot suave, Fly, Speed, Hitbox "Skeleton", Minimizar, e light layout.
--]]

local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local RunService = game:GetService("RunService")
local CoreGui = game:GetService("CoreGui")
local localPlayer = Players.LocalPlayer

-- Paleta de cores pastel / moderna
local colors = {
    bg = Color3.fromRGB(40,44,60),
    accent = Color3.fromRGB(103,146,255),
    btn = Color3.fromRGB(93,108,198),
    btn_hover = Color3.fromRGB(123,158,255),
    text = Color3.fromRGB(221,234,255),
    shadow = Color3.fromRGB(30,30,40),
}

------------------------------------------------------------------
---------------------- GUI DEFINITIONS ---------------------------
------------------------------------------------------------------
local hubGui = Instance.new("ScreenGui")
hubGui.Name = "RaioVerseHub"
hubGui.Parent = CoreGui

-- Sombra
local shadowFrame = Instance.new("Frame", hubGui)
shadowFrame.BackgroundColor3 = colors.shadow
shadowFrame.BackgroundTransparency = 0.5
shadowFrame.Size = UDim2.new(0,340,0,240)
shadowFrame.Position = UDim2.new(0.5,-172,0.5,-122)
shadowFrame.ZIndex = 0
Instance.new("UICorner", shadowFrame).CornerRadius = UDim.new(0,17)

local mainFrame = Instance.new("Frame", hubGui)
mainFrame.Size = UDim2.new(0,340,0,240)
mainFrame.Position = UDim2.new(0.5,-170,0.5,-120)
mainFrame.BackgroundColor3 = colors.bg
mainFrame.BackgroundTransparency = 0.08
mainFrame.BorderSizePixel = 0
mainFrame.ZIndex = 1
Instance.new("UICorner", mainFrame).CornerRadius = UDim.new(0,17)
mainFrame.Active, mainFrame.Draggable, mainFrame.Visible = true, true, true

local title = Instance.new("TextLabel", mainFrame)
title.Size = UDim2.new(1, 0, 0, 38)
title.Position = UDim2.new(0,0,0,0)
title.BackgroundTransparency = 1
title.Font = Enum.Font.GothamBold
title.Text = "RaioVerse Hub"
title.TextColor3 = colors.accent
title.TextSize = 26
title.ZIndex = 2

local minBtn = Instance.new("TextButton", mainFrame)
minBtn.Size = UDim2.new(0,31,0,31)
minBtn.Position = UDim2.new(1,-40,0,7)
minBtn.BackgroundColor3 = colors.btn
minBtn.Text = "—"
minBtn.Font = Enum.Font.GothamSemibold
minBtn.TextColor3 = colors.text
minBtn.TextSize = 24
minBtn.ZIndex = 3
Instance.new("UICorner", minBtn).CornerRadius = UDim.new(0,11)

local btnFrame = Instance.new("Frame", mainFrame)
btnFrame.Size = UDim2.new(1,-30,1,-55)
btnFrame.Position = UDim2.new(0,15,0,45)
btnFrame.BackgroundTransparency = 1
btnFrame.ZIndex = 3
local btnList = Instance.new("UIListLayout", btnFrame)
btnList.Padding = UDim.new(0,11)
btnList.HorizontalAlignment = Enum.HorizontalAlignment.Center
btnList.SortOrder = Enum.SortOrder.LayoutOrder

local credits = Instance.new("TextLabel", mainFrame)
credits.Size = UDim2.new(1,0,0,17)
credits.Position = UDim2.new(0,0,1,-19)
credits.BackgroundTransparency = 1
credits.Text = "☄️ RaioVerse"
credits.TextColor3 = colors.accent
credits.Font = Enum.Font.GothamSemibold
credits.TextSize = 13
credits.ZIndex = 3

-- Minimizado
local iconMin = Instance.new("ImageButton", hubGui)
iconMin.AutoButtonColor = true
iconMin.Size = UDim2.new(0,32,0,32)
iconMin.Position = UDim2.new(0,22,0,22)
iconMin.BackgroundTransparency = 0.5
iconMin.Image = "rbxassetid://3926305904" -- Ícone circulo/bolinha (pode trocar por outros)
iconMin.Visible = false
iconMin.ZIndex = 10

------------------------------------------------------------------
---------------------- TOGGLE LOGIC ------------------------------
------------------------------------------------------------------
local state = {
    AimBot = false,
    Fly = false,
    Speed = false,
    Hitbox = false
}

-- Botão estilizado
local function makeToggleBtn(txt)
    local btn = Instance.new("TextButton", btnFrame)
    btn.Size = UDim2.new(1,0,0,38)
    btn.BackgroundColor3 = colors.btn
    btn.Text = "[ OFF ]  " .. txt
    btn.Font = Enum.Font.Gotham
    btn.TextColor3 = colors.text
    btn.TextSize = 19
    btn.ZIndex = 4
    Instance.new("UICorner",btn).CornerRadius = UDim.new(0,10)
    btn.MouseEnter:Connect(function() btn.BackgroundColor3 = colors.btn_hover end)
    btn.MouseLeave:Connect(function() btn.BackgroundColor3 = colors.btn end)
    return btn
end

------------------------------------------------------------------
----- [AimBot Suave] ---------------------------------------------
------------------------------------------------------------------
local aimbotConn = nil
local AIMBOT_SMOOTHNESS = 0.12

local function getClosestPlayer()
    local smallest = math.huge
    local cam = workspace.CurrentCamera
    local bestPlayer = nil
    for _,p in pairs(Players:GetPlayers()) do
        if p ~= localPlayer and p.Character and p.Character:FindFirstChild("HumanoidRootPart") and p.Character:FindFirstChild("Humanoid")
            and p.Character.Humanoid.Health > 0 then
            local hrp = p.Character.HumanoidRootPart.Position
            local pos, onScreen = cam:WorldToViewportPoint(hrp)
            if onScreen then
                local dist = (Vector2.new(pos.X, pos.Y) - Vector2.new(cam.ViewportSize.X/2, cam.ViewportSize.Y/2)).Magnitude
                if dist < smallest then
                    smallest = dist
                    bestPlayer = p
                end
            end
        end
    end
    return bestPlayer
end

local function aimBotStep()
    if state.AimBot then
        local cam = workspace.CurrentCamera
        local currentCF = cam.CFrame
        local targetPlayer = getClosestPlayer()
        if targetPlayer and targetPlayer.Character and targetPlayer.Character:FindFirstChild("HumanoidRootPart") then
            local hrp = targetPlayer.Character.HumanoidRootPart
            local lookAt = hrp.Position
            local desiredCF = CFrame.new(cam.CFrame.Position, lookAt)
            cam.CFrame = currentCF:Lerp(desiredCF, AIMBOT_SMOOTHNESS)
        end
    end
end
------------------------------------------------------------------
----- [Fly] ------------------------------------------------------
------------------------------------------------------------------
local flyConn = nil
local FlySpeed = 60
local function flyActivate()
    if not localPlayer.Character or not localPlayer.Character:FindFirstChild("HumanoidRootPart") then return end
    local char = localPlayer.Character
    local hrp = char:FindFirstChild("HumanoidRootPart")
    local flyT = Instance.new("BodyVelocity", hrp)
    flyT.MaxForce = Vector3.new(1, 1, 1) * 1e5
    flyT.Name = "RaioFlyForce"
    flyConn = RunService.RenderStepped:Connect(function()
        local flyVec = Vector3.new()
        if UserInputService:IsKeyDown(Enum.KeyCode.W) then flyVec = flyVec + workspace.CurrentCamera.CFrame.LookVector end
        if UserInputService:IsKeyDown(Enum.KeyCode.S) then flyVec = flyVec - workspace.CurrentCamera.CFrame.LookVector end
        if UserInputService:IsKeyDown(Enum.KeyCode.A) then flyVec = flyVec - workspace.CurrentCamera.CFrame.RightVector end
        if UserInputService:IsKeyDown(Enum.KeyCode.D) then flyVec = flyVec + workspace.CurrentCamera.CFrame.RightVector end
        if UserInputService:IsKeyDown(Enum.KeyCode.Space) then flyVec = flyVec + Vector3.new(0,1,0) end
        if UserInputService:IsKeyDown(Enum.KeyCode.LeftControl) then flyVec = flyVec - Vector3.new(0,1,0) end
        flyT.Velocity = flyVec.Magnitude > 0 and flyVec.Unit * FlySpeed or Vector3.new(0,0,0)
    end)
end
local function stopFly()
    if flyConn then flyConn:Disconnect() flyConn = nil end
    if localPlayer.Character and localPlayer.Character:FindFirstChild("HumanoidRootPart") then
        for _,obj in ipairs(localPlayer.Character.HumanoidRootPart:GetChildren()) do
            if obj.Name == "RaioFlyForce" then obj:Destroy() end
        end
    end
end

------------------------------------------------------------------
----- [Speed] ----------------------------------------------------
------------------------------------------------------------------
local SpeedValue = 45
local function setSpeed(enabled)
    if localPlayer.Character and localPlayer.Character:FindFirstChildWhichIsA("Humanoid") then
        localPlayer.Character:FindFirstChildWhichIsA("Humanoid").WalkSpeed = enabled and SpeedValue or 16
    end
end
local function FixSpeed() setSpeed(state.Speed) end
localPlayer.CharacterAdded:Connect(function() wait(0.3) FixSpeed() end)

RunService.Stepped:Connect(function() if state.Speed then setSpeed(true) end end)

------------------------------------------------------------------
----- [Hitbox Skeleton] ------------------------------------------
------------------------------------------------------------------
-- Função "draw skeleton" usando linhas (Attachs): entre cabeça, torso, braços, pés dos outros players
local skeletons = {}
local bones = {
    -- {de, para}
    {"Head","UpperTorso"},
    {"UpperTorso","LowerTorso"},
    {"UpperTorso","LeftUpperArm"}, {"LeftUpperArm","LeftLowerArm"}, {"LeftLowerArm","LeftHand"},
    {"UpperTorso","RightUpperArm"}, {"RightUpperArm","RightLowerArm"}, {"RightLowerArm","RightHand"},
    {"LowerTorso","LeftUpperLeg"}, {"LeftUpperLeg","LeftLowerLeg"}, {"LeftLowerLeg","LeftFoot"},
    {"LowerTorso","RightUpperLeg"}, {"RightUpperLeg","RightLowerLeg"}, {"RightLowerLeg","RightFoot"},
}

local function clearSkeletons()
    for _,tab in ipairs(skeletons) do for _,a in ipairs(tab) do pcall(function() a:Destroy() end) end end
    table.clear(skeletons)
end

local function updateSkeletons()
    clearSkeletons()
    if not state.Hitbox then return end
    for _,plr in ipairs(Players:GetPlayers()) do
        if plr ~= localPlayer and plr.Character then
            local nodes = {}
            for _,bone in ipairs(bones) do
                local partA = plr.Character:FindFirstChild(bone[1])
                local partB = plr.Character:FindFirstChild(bone[2])
                if partA and partB then
                    local att = Instance.new("Attachment")
                    att.Parent = partA
                    local att2 = Instance.new("Attachment")
                    att2.Parent = partB
                    local line = Instance.new("Beam")
                    line.Attachment0, line.Attachment1 = att, att2
                    line.FaceCamera = true
                    line.Color = ColorSequence.new(colors.accent)
                    line.Width0 = 0.23; line.Width1 = 0.09
                    line.Transparency = NumberSequence.new(0.18)
                    line.LightEmission = 0.56
                    line.Parent = partA
                    table.insert(nodes, att)
                    table.insert(nodes, att2)
                    table.insert(nodes, line)
                end
            end
            table.insert(skeletons, nodes)
        end
    end
end

-- Real-time skeleton update!
local skeletonConn = nil
local function toggleSkeleton(active)
    if active and not skeletonConn then
        skeletonConn = RunService.RenderStepped:Connect(function()
            updateSkeletons()
        end)
    elseif not active and skeletonConn then
        skeletonConn:Disconnect()
        skeletonConn = nil
        clearSkeletons()
    end
end

------------------------------------------------------------------
---------------------- BOTÕES DO HUB -----------------------------
------------------------------------------------------------------
local aimBtn = makeToggleBtn("AimBot Suave")
aimBtn.MouseButton1Click:Connect(function()
    state.AimBot = not state.AimBot
    aimBtn.Text = (state.AimBot and "[ ON ] " or "[ OFF ] ") .. "AimBot Suave"
    if state.AimBot and not aimbotConn then
        aimbotConn = RunService.RenderStepped:Connect(aimBotStep)
    elseif not state.AimBot and aimbotConn then
        aimbotConn:Disconnect()
        aimbotConn = nil
    end
end)

local flyBtn = makeToggleBtn("Fly")
flyBtn.MouseButton1Click:Connect(function()
    state.Fly = not state.Fly
    flyBtn.Text = (state.Fly and "[ ON ] " or "[ OFF ] ") .. "Fly"
    if state.Fly then
        flyActivate()
    else
        stopFly()
    end
end)

local speedBtn = makeToggleBtn("Velocidade Turbo")
speedBtn.MouseButton1Click:Connect(function()
    state.Speed = not state.Speed
    speedBtn.Text = (state.Speed and "[ ON ] " or "[ OFF ] ") .. "Velocidade Turbo"
    setSpeed(state.Speed)
end)

local hitboxBtn = makeToggleBtn("Skeleton Hitbox")
hitboxBtn.MouseButton1Click:Connect(function()
    state.Hitbox = not state.Hitbox
    hitboxBtn.Text = (state.Hitbox and "[ ON ] " or "[ OFF ] ") .. "Skeleton Hitbox"
    toggleSkeleton(state.Hitbox)
end)

------------------------------------------------------------------
---- MINIMIZAR/ABRIR HUB com icone discreto ----------------------
------------------------------------------------------------------
local minimized = false
minBtn.MouseButton1Click:Connect(function()
    minimized = true
    mainFrame.Visible = false
    shadowFrame.Visible = false
    iconMin.Visible = true
end)
iconMin.MouseButton1Click:Connect(function()
    minimized = false
    mainFrame.Visible = true
    shadowFrame.Visible = true
    iconMin.Visible = false
end)

hubGui.DisplayOrder = 8926

------------------------------------------------------------------
------ Mais ideias para seu HUB: (adicionar facilmente!) ----------
------------------------------------------------------------------
--[[
-- ESP: Mostra nome/distância dos jogadores acima da cabeça
-- NoClip: Libera atravessar paredes
-- JumpPower: Ajusta altura do pulo
-- Super Knockback: Empurra mais forte ao atacar
-- ClickTP: Teleportar para onde clicar
-- FOV Changer: Muda campo de visão
-- Color Theme: Troca esquema de cor do hub
-- Anti-AFK: Evita kick de inatividade
-- E muito mais!
]]

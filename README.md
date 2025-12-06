--[[
    RaioVerse Hub - Blox Fruits (Com Minimizar!)
    Funções: AimBot suave, Velocidade Turbo (personagem), Hitbox Gigante, Minimizar/Restaurar
--]]

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local CoreGui = game:GetService("CoreGui")
local localPlayer = Players.LocalPlayer

local colors = {
    bg = Color3.fromRGB(40,44,60),
    accent = Color3.fromRGB(103,146,255),
    btn = Color3.fromRGB(93,108,198),
    btn_hover = Color3.fromRGB(123,158,255),
    text = Color3.fromRGB(221,234,255),
}

local state = {
    AimBot = false,
    Speed = false,
    HitboxGigante = false
}

-- GUI
local hubGui = Instance.new("ScreenGui", CoreGui)
hubGui.Name = "RaioVerseHub"

local mainFrame = Instance.new("Frame", hubGui)
mainFrame.Size = UDim2.new(0,310,0,190)
mainFrame.Position = UDim2.new(0.5,-155,0.5,-95)
mainFrame.BackgroundColor3 = colors.bg
mainFrame.BackgroundTransparency = 0.08
mainFrame.BorderSizePixel = 0
Instance.new("UICorner", mainFrame).CornerRadius = UDim.new(0,17)
mainFrame.Active = true
mainFrame.Draggable = true

local title = Instance.new("TextLabel", mainFrame)
title.Size = UDim2.new(1,0,0,32)
title.Position = UDim2.new(0,0,0,0)
title.BackgroundTransparency = 1
title.Font = Enum.Font.GothamBold
title.Text = "RaioVerse Hub"
title.TextColor3 = colors.accent
title.TextSize = 21

-- Minimizar button (top right)
local minBtn = Instance.new("TextButton", mainFrame)
minBtn.Size = UDim2.new(0,28,0,28)
minBtn.Position = UDim2.new(1,-36,0,5)
minBtn.BackgroundColor3 = colors.btn
minBtn.Text = "—"
minBtn.Font = Enum.Font.GothamBold
minBtn.TextColor3 = colors.text
minBtn.TextSize = 21
Instance.new("UICorner", minBtn).CornerRadius = UDim.new(0,9)

-- Minimizado (bolinha avatar)
local iconMin = Instance.new("ImageButton", hubGui)
iconMin.Size = UDim2.new(0,26,0,26)
iconMin.Position = UDim2.new(0,16,0,16)
iconMin.BackgroundTransparency = 0.5
iconMin.Image = "rbxassetid://3926305904"
iconMin.Visible = false
iconMin.ZIndex = 10

-- Minimizar logic
minBtn.MouseButton1Click:Connect(function()
    mainFrame.Visible = false
    iconMin.Visible = true
end)
iconMin.MouseButton1Click:Connect(function()
    mainFrame.Visible = true
    iconMin.Visible = false
end)


-- Buttons container
local btnFrame = Instance.new("Frame", mainFrame)
btnFrame.Size = UDim2.new(1,-30,1,-46)
btnFrame.Position = UDim2.new(0,15,0,38)
btnFrame.BackgroundTransparency = 1
local btnList = Instance.new("UIListLayout", btnFrame)
btnList.Padding = UDim.new(0,12)
btnList.HorizontalAlignment = Enum.HorizontalAlignment.Center
btnList.SortOrder = Enum.SortOrder.LayoutOrder

local function makeToggleBtn(txt)
    local btn = Instance.new("TextButton", btnFrame)
    btn.Size = UDim2.new(1,0,0,36)
    btn.BackgroundColor3 = colors.btn
    btn.Text = "[ OFF ]  "..txt
    btn.Font = Enum.Font.Gotham
    btn.TextColor3 = colors.text
    btn.TextSize = 17
    Instance.new("UICorner",btn).CornerRadius = UDim.new(0,9)
    btn.MouseEnter:Connect(function() btn.BackgroundColor3 = colors.btn_hover end)
    btn.MouseLeave:Connect(function() btn.BackgroundColor3 = colors.btn end)
    return btn
end

------------------------------------------
-- FUNÇÃO: HITBOX GIGANTE (Blox Fruits)
------------------------------------------
local hitboxConn = nil
local HITBOX_SIZE = Vector3.new(90,90,90)

local function setHitboxSize(part, size)
    if part and part:IsA("BasePart") then
        part.Size = size
        part.Transparency = 0.93
        part.CanCollide = false
        part.Massless = true
    end
end
local function resetHitboxSize(part)
    if part and part:IsA("BasePart") then
        part.Size = Vector3.new(1,1,2)
        part.Transparency = 1
        part.CanCollide = false
        part.Massless = true
    end
end

local function applyGigante()
    local char = localPlayer.Character
    if not char then return end
    local tool = char:FindFirstChildOfClass("Tool")
    if tool then
        for _,v in ipairs(tool:GetDescendants()) do
            if v:IsA("BasePart") then
                setHitboxSize(v, HITBOX_SIZE)
            end
        end
    end
    for _,v in ipairs(char:GetChildren()) do
        if v:IsA("BasePart") and (v.Name:lower():find("hand") or v.Name:lower():find("leg") or v.Name:lower():find("arm") or v.Name:lower():find("foot")) then
            setHitboxSize(v, HITBOX_SIZE)
        end
    end
end
local function revertGigante()
    local char = localPlayer.Character
    if not char then return end
    local tool = char:FindFirstChildOfClass("Tool")
    if tool then
        for _,v in ipairs(tool:GetDescendants()) do
            if v:IsA("BasePart") then
                resetHitboxSize(v)
            end
        end
    end
    for _,v in ipairs(char:GetChildren()) do
        if v:IsA("BasePart") and (v.Name:lower():find("hand") or v.Name:lower():find("leg") or v.Name:lower():find("arm") or v.Name:lower():find("foot")) then
            resetHitboxSize(v)
        end
    end
end

local function startHitboxGigante()
    if hitboxConn then return end
    hitboxConn = RunService.RenderStepped:Connect(function()
        if state.HitboxGigante then applyGigante() else revertGigante() end
    end)
end
local function stopHitboxGigante()
    if hitboxConn then hitboxConn:Disconnect(); hitboxConn = nil end
    revertGigante()
end

------------------------------------------
-- AIMBOT SUAVE (mira suavemente no player mais próximo)
------------------------------------------
local aimbotConn = nil
local AIMBOT_SMOOTHNESS = 0.13
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
            cam.CFrame = currentCF:Lerp(CFrame.new(cam.CFrame.Position, lookAt), AIMBOT_SMOOTHNESS)
        end
    end
end

------------------------------------------
-- SPEED (CORRER MAIS RÁPIDO)
------------------------------------------
local SpeedValue = 45
local function setSpeed(enabled)
    if localPlayer.Character and localPlayer.Character:FindFirstChildWhichIsA("Humanoid") then
        localPlayer.Character:FindFirstChildWhichIsA("Humanoid").WalkSpeed = enabled and SpeedValue or 16
    end
end
local function FixSpeed() setSpeed(state.Speed) end
localPlayer.CharacterAdded:Connect(function() wait(0.3) FixSpeed() end)
RunService.Stepped:Connect(function() if state.Speed then setSpeed(true) end end)

------------------------------------------
-- BOTÕES
------------------------------------------
local aimBtn = makeToggleBtn("AimBot Suave")
aimBtn.MouseButton1Click:Connect(function()
    state.AimBot = not state.AimBot
    aimBtn.Text = (state.AimBot and "[ ON ] " or "[ OFF ] ").."AimBot Suave"
    if state.AimBot and not aimbotConn then
        aimbotConn = RunService.RenderStepped:Connect(aimBotStep)
    elseif not state.AimBot and aimbotConn then
        aimbotConn:Disconnect(); aimbotConn = nil
    end
end)

local speedBtn = makeToggleBtn("Velocidade Turbo (Personagem)")
speedBtn.MouseButton1Click:Connect(function()
    state.Speed = not state.Speed
    speedBtn.Text = (state.Speed and "[ ON ] " or "[ OFF ] ").."Velocidade Turbo (Personagem)"
    setSpeed(state.Speed)
end)

local hitboxBtn = makeToggleBtn("HITBOX GIGANTE")
hitboxBtn.MouseButton1Click:Connect(function()
    state.HitboxGigante = not state.HitboxGigante
    hitboxBtn.Text = (state.HitboxGigante and "[ ON ] " or "[ OFF ] ").."HITBOX GIGANTE"
    if state.HitboxGigante then
        startHitboxGigante()
    else
        stopHitboxGigante()
    end
end)

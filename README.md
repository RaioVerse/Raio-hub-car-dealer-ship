--[[
    RaioVerse Hub - Jogos de Corrida + Auto Win CDT!
    Funções: AimBot suave, Velocidade Turbo, ESP Head, AutoWin CDT, minimização simples
--]]

local Players = game:GetService("Players")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local UserInputService = game:GetService("UserInputService")
local RunService = game:GetService("RunService")
local CoreGui = game:GetService("CoreGui")
local localPlayer = Players.LocalPlayer

local colors = {
    bg = Color3.fromRGB(40,44,60),
    accent = Color3.fromRGB(103,146,255),
    btn = Color3.fromRGB(93,108,198),
    btn_hover = Color3.fromRGB(123,158,255),
    text = Color3.fromRGB(221,234,255),
    shadow = Color3.fromRGB(30,30,40),
}

-- HUB GUI
local hubGui = Instance.new("ScreenGui")
hubGui.Name = "RaioVerseHub"
hubGui.Parent = CoreGui

-- Sombra frame
local shadowFrame = Instance.new("Frame", hubGui)
shadowFrame.BackgroundColor3 = colors.shadow
shadowFrame.BackgroundTransparency = 0.5
shadowFrame.Size = UDim2.new(0,340,0,235)
shadowFrame.Position = UDim2.new(0.5,-172,0.5,-117)
shadowFrame.ZIndex = 0
Instance.new("UICorner", shadowFrame).CornerRadius = UDim.new(0,17)

local mainFrame = Instance.new("Frame", hubGui)
mainFrame.Size = UDim2.new(0,340,0,235)
mainFrame.Position = UDim2.new(0.5,-170,0.5,-115)
mainFrame.BackgroundColor3 = colors.bg
mainFrame.BackgroundTransparency = 0.08
mainFrame.BorderSizePixel = 0
mainFrame.ZIndex = 1
Instance.new("UICorner", mainFrame).CornerRadius = UDim.new(0,17)
mainFrame.Active, mainFrame.Draggable, mainFrame.Visible = true, true, true

local title = Instance.new("TextLabel", mainFrame)
title.Size = UDim2.new(1, 0, 0, 36)
title.Position = UDim2.new(0,0,0,0)
title.BackgroundTransparency = 1
title.Font = Enum.Font.GothamBold
title.Text = "RaioVerse Hub"
title.TextColor3 = colors.accent
title.TextSize = 24
title.ZIndex = 2

local minBtn = Instance.new("TextButton", mainFrame)
minBtn.Size = UDim2.new(0,31,0,31)
minBtn.Position = UDim2.new(1,-40,0,7)
minBtn.BackgroundColor3 = colors.btn
minBtn.Text = "—"
minBtn.Font = Enum.Font.GothamSemibold
minBtn.TextColor3 = colors.text
minBtn.TextSize = 22
minBtn.ZIndex = 3
Instance.new("UICorner", minBtn).CornerRadius = UDim.new(0,11)

local btnFrame = Instance.new("Frame", mainFrame)
btnFrame.Size = UDim2.new(1,-30,1,-55)
btnFrame.Position = UDim2.new(0,15,0,40)
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
iconMin.Size = UDim2.new(0,32,0,32)
iconMin.Position = UDim2.new(0,22,0,22)
iconMin.BackgroundTransparency = 0.5
iconMin.Image = "rbxassetid://3926305904" -- círculo discreto
iconMin.Visible = false
iconMin.ZIndex = 10

------------------------------------------------------------------
---------------------- TOGGLE LOGIC ------------------------------
------------------------------------------------------------------
local state = {
    AimBot = false,
    Speed = false,
    Hitbox = false
}

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
----- 1. AimBot Suave --------------------------------------------
------------------------------------------------------------------
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
------------------------------------------------------------------
----- 2. Speed Turbo ---------------------------------------------
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
----- 3. Head Hitbox ESP -----------------------------------------
------------------------------------------------------------------
local headHighlights = {}

local function clearHeadHighlights()
    for _,h in ipairs(headHighlights) do pcall(function() h:Destroy() end) end
    table.clear(headHighlights)
end

local function updateHeadHighlights()
    clearHeadHighlights()
    if not state.Hitbox then return end
    for _,plr in ipairs(Players:GetPlayers()) do
        if plr ~= localPlayer and plr.Character and plr.Character:FindFirstChild("Head") then
            local adorn = Instance.new("BoxHandleAdornment")
            adorn.Adornee = plr.Character.Head
            adorn.Size = Vector3.new(1.6,1.1,1.8)
            adorn.Color3 = colors.accent
            adorn.AlwaysOnTop = true
            adorn.Transparency = 0.2
            adorn.ZIndex = 7
            adorn.Parent = workspace
            table.insert(headHighlights, adorn)
        end
    end
end

local headConn = nil
local function toggleHeadESP(active)
    if active and not headConn then
        headConn = RunService.RenderStepped:Connect(function()
            updateHeadHighlights()
        end)
    elseif not active and headConn then
        headConn:Disconnect()
        headConn = nil
        clearHeadHighlights()
    end
end

------------------------------------------------------------------
----- 4. Auto Win CDT --------------------------------------------
------------------------------------------------------------------
local function AutoWinCDT()
    local eventsFolder = ReplicatedStorage:FindFirstChild("Events")
    if not eventsFolder then return end
    local checkpointsEvent = eventsFolder:FindFirstChild("RaceCheckpoint")
    local finishEvent     = eventsFolder:FindFirstChild("FinishRace")
    if not (checkpointsEvent and finishEvent) then return end

    -- Você pode ajustar 25 para a quantidade da corrida desejada!
    for num=1,25 do
        checkpointsEvent:FireServer(num, localPlayer)
        wait(0.15)
    end
    -- Terminando a corrida
    finishEvent:FireServer(localPlayer)
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

local speedBtn = makeToggleBtn("Velocidade Turbo")
speedBtn.MouseButton1Click:Connect(function()
    state.Speed = not state.Speed
    speedBtn.Text = (state.Speed and "[ ON ] " or "[ OFF ] ") .. "Velocidade Turbo"
    setSpeed(state.Speed)
end)

local headBtn = makeToggleBtn("ESP Head Hitbox")
headBtn.MouseButton1Click:Connect(function()
    state.Hitbox = not state.Hitbox
    headBtn.Text = (state.Hitbox and "[ ON ] " or "[ OFF ] ") .. "ESP Head Hitbox"
    toggleHeadESP(state.Hitbox)
end)

local autoWinBtn = Instance.new("TextButton", btnFrame)
autoWinBtn.Size = UDim2.new(1,0,0,38)
autoWinBtn.BackgroundColor3 = Color3.fromRGB(124, 185, 77)
autoWinBtn.Text = "Auto Win CDT"
autoWinBtn.Font = Enum.Font.GothamBold
autoWinBtn.TextColor3 = colors.text
autoWinBtn.TextSize = 19
autoWinBtn.ZIndex = 4
Instance.new("UICorner", autoWinBtn).CornerRadius = UDim.new(0,10)
autoWinBtn.MouseEnter:Connect(function() autoWinBtn.BackgroundColor3 = Color3.fromRGB(134,205,97) end)
autoWinBtn.MouseLeave:Connect(function() autoWinBtn.BackgroundColor3 = Color3.fromRGB(124,185,77) end)

autoWinBtn.MouseButton1Click:Connect(function()
    pcall(AutoWinCDT)
end)

------------------------------------------------------------------
---- MINIMIZAR/ABRIR HUB (bolinha discreta) ----------------------
------------------------------------------------------------------
minBtn.MouseButton1Click:Connect(function()
    mainFrame.Visible = false
    shadowFrame.Visible = false
    iconMin.Visible = true
end)
iconMin.MouseButton1Click:Connect(function()
    mainFrame.Visible = true
    shadowFrame.Visible = true
    iconMin.Visible = false
end)

hubGui.DisplayOrder = 8926

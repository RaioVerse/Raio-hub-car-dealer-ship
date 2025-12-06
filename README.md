--[[
    RaioVerse Hub - Auto Click + Ataque à Distância (Blox Fruits)
    Funções: AimBot suave, Velocidade Turbo, ESP Head, Turbo Carro, Auto Click, Atacar NPCs de longe
--]]

local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local RunService = game:GetService("RunService")
local CoreGui = game:GetService("CoreGui")
local Workspace = game:GetService("Workspace")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local localPlayer = Players.LocalPlayer

local colors = {
    bg = Color3.fromRGB(40,44,60),
    accent = Color3.fromRGB(103,146,255),
    btn = Color3.fromRGB(93,108,198),
    btn_hover = Color3.fromRGB(123,158,255),
    text = Color3.fromRGB(221,234,255),
    shadow = Color3.fromRGB(30,30,40),
}
local state = {
    AimBot = false,
    Speed = false,
    Hitbox = false,
    AutoClick = false,
    AutoAttackNPC = false
}

-- GUI Layout
local hubGui = Instance.new("ScreenGui", CoreGui)
hubGui.Name = "RaioVerseHub"

local shadowFrame = Instance.new("Frame", hubGui)
shadowFrame.BackgroundColor3 = colors.shadow
shadowFrame.BackgroundTransparency = 0.5
shadowFrame.Size = UDim2.new(0,355,0,300)
shadowFrame.Position = UDim2.new(0.5,-177,0.5,-148)
shadowFrame.ZIndex = 0
Instance.new("UICorner", shadowFrame).CornerRadius = UDim.new(0,17)

local mainFrame = Instance.new("Frame", hubGui)
mainFrame.Size = UDim2.new(0,355,0,300)
mainFrame.Position = UDim2.new(0.5,-175,0.5,-150)
mainFrame.BackgroundColor3 = colors.bg
mainFrame.BackgroundTransparency = 0.08
mainFrame.BorderSizePixel = 0
mainFrame.ZIndex = 1
Instance.new("UICorner", mainFrame).CornerRadius = UDim.new(0,17)
mainFrame.Active = true
mainFrame.Draggable = true

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

local iconMin = Instance.new("ImageButton", hubGui)
iconMin.Size = UDim2.new(0,32,0,32)
iconMin.Position = UDim2.new(0,22,0,22)
iconMin.BackgroundTransparency = 0.5
iconMin.Image = "rbxassetid://3926305904" -- círculo discreto
iconMin.Visible = false
iconMin.ZIndex = 10

------------------------------------------------------------------------
-- Botão padrão estilizado
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

------------------------------------------------------------------------
-- 1. AimBot Suave
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

------------------------------------------------------------------------
-- 2. Velocidade Turbo (Personagem)
local SpeedValue = 45
local function setSpeed(enabled)
    if localPlayer.Character and localPlayer.Character:FindFirstChildWhichIsA("Humanoid") then
        localPlayer.Character:FindFirstChildWhichIsA("Humanoid").WalkSpeed = enabled and SpeedValue or 16
    end
end
local function FixSpeed() setSpeed(state.Speed) end
localPlayer.CharacterAdded:Connect(function() wait(0.3) FixSpeed() end)
RunService.Stepped:Connect(function() if state.Speed then setSpeed(true) end end)

------------------------------------------------------------------------
-- 3. ESP Head Hitbox
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

------------------------------------------------------------------------
-- 4. Turbo do Carro (igual versão anterior)
local turboBtnCooldown = false
local TURBO_DURATION = 3.2
local TURBO_FORCE = 2500
local function getPlayerCar()
    for _,v in ipairs(Workspace:GetChildren()) do
        if v:IsA("Model") and v:FindFirstChildOfClass("VehicleSeat") and v.Name:lower():find(localPlayer.Name:lower()) then
            return v:FindFirstChildOfClass("VehicleSeat"), v
        end
    end
    if localPlayer.Character then
        local seat = localPlayer.Character:FindFirstChildOfClass("VehicleSeat")
        if seat then return seat, localPlayer.Character end
    end
    return nil, nil
end
local function activateCarTurbo()
    if turboBtnCooldown then return end
    turboBtnCooldown = true
    local seat, carModel = getPlayerCar()
    if seat then
        local turbo = Instance.new("BodyVelocity")
        turbo.MaxForce = Vector3.new(1,1,1) * 1e6
        turbo.Velocity = seat.CFrame.LookVector * TURBO_FORCE
        turbo.Parent = seat
        turbo.Name = "RaioTurbo"
        if carModel and carModel:IsA("Model") then
            for _,part in ipairs(carModel:GetDescendants()) do
                if part:IsA("BasePart") then
                    part.Color = Color3.fromRGB(106,255,133)
                end
            end
        end
        wait(TURBO_DURATION)
        turbo:Destroy()
        if carModel and carModel:IsA("Model") then
            for _,part in ipairs(carModel:GetDescendants()) do
                if part:IsA("BasePart") then
                    part.Color = Color3.fromRGB(255,255,255)
                end
            end
        end
    else
        warn("Carro não localizado. Entre em seu carro antes de ativar o Turbo!")
    end
    turboBtnCooldown = false
end

------------------------------------------------------------------------
-- 5. **AUTO CLICK** ---------------------------------------------------
local autoClickConn = nil
local function startAutoClick()
    if autoClickConn then return end
    autoClickConn = RunService.RenderStepped:Connect(function()
        if UserInputService.MouseEnabled and state.AutoClick then
            mouse1click()
        end
    end)
end
local function stopAutoClick()
    if autoClickConn then autoClickConn:Disconnect() autoClickConn = nil end
end

-- Função universal de mouse1click (usada por executores), para roblox executor padrão:
function mouse1click()
    localVirtualInput = game:GetService("VirtualInputManager")
    localVirtualInput:SendMouseButtonEvent(0,0,0,true,game,0)
    localVirtualInput:SendMouseButtonEvent(0,0,0,false,game,0)
end

------------------------------------------------------------------------
-- 6. **Ataque à Distância NPCs Blox Fruits**
local autoAttackConn = nil
local function attackNPCsBloxFruit()
    local tool = localPlayer.Character and localPlayer.Character:FindFirstChildOfClass("Tool")
    if tool then
        for _,npc in ipairs(Workspace.Enemies:GetChildren()) do
            if npc:FindFirstChild("Humanoid") and npc.Humanoid.Health > 0 then
                local args = {
                    [1] = npc.Humanoid,
                    [2] = tool
                }
                -- Usa evento remoto padrão de ataques (editável conforme armas do BF)
                pcall(function()
                    tool:Activate()
                    tool:Activate() -- duplo ataque por garantia
                    -- Algumas frutas/ferramentas usam remote: ReplicatedStorage.Remotes.CommF_:InvokeServer(...)
                end)
            end
        end
    end
end
local function startAutoAttackNPC()
    if autoAttackConn then return end
    autoAttackConn = RunService.RenderStepped:Connect(function()
        if state.AutoAttackNPC then
            attackNPCsBloxFruit()
        end
    end)
end
local function stopAutoAttackNPC()
    if autoAttackConn then autoAttackConn:Disconnect() autoAttackConn = nil end
end

------------------------------------------------------------------------
-- Botões
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

local speedBtn = makeToggleBtn("Velocidade Turbo (Personagem)")
speedBtn.MouseButton1Click:Connect(function()
    state.Speed = not state.Speed
    speedBtn.Text = (state.Speed and "[ ON ] " or "[ OFF ] ") .. "Velocidade Turbo (Personagem)"
    setSpeed(state.Speed)
end)

local headBtn = makeToggleBtn("ESP Head Hitbox")
headBtn.MouseButton1Click:Connect(function()
    state.Hitbox = not state.Hitbox
    headBtn.Text = (state.Hitbox and "[ ON ] " or "[ OFF ] ") .. "ESP Head Hitbox"
    toggleHeadESP(state.Hitbox)
end)

local turboBtn = Instance.new("TextButton", btnFrame)
turboBtn.Size = UDim2.new(1,0,0,38)
turboBtn.BackgroundColor3 = Color3.fromRGB(118,210,98)
turboBtn.Text = "Turbo do Carro"
turboBtn.Font = Enum.Font.GothamBold
turboBtn.TextColor3 = colors.text
turboBtn.TextSize = 19
turboBtn.ZIndex = 4
Instance.new("UICorner", turboBtn).CornerRadius = UDim.new(0,10)
turboBtn.MouseEnter:Connect(function() turboBtn.BackgroundColor3 = Color3.fromRGB(149,242,110) end)
turboBtn.MouseLeave:Connect(function() turboBtn.BackgroundColor3 = Color3.fromRGB(118,210,98) end)
turboBtn.MouseButton1Click:Connect(function()
    activateCarTurbo()
end)

local autoClickBtn = makeToggleBtn("Auto Click")
autoClickBtn.MouseButton1Click:Connect(function()
    state.AutoClick = not state.AutoClick
    autoClickBtn.Text = (state.AutoClick and "[ ON ] " or "[ OFF ] ") .. "Auto Click"
    if state.AutoClick then
        startAutoClick()
    else
        stopAutoClick()
    end
end)

local autoAttackBtn = makeToggleBtn("Ataque à distância NPCs (Blox Fruits)")
autoAttackBtn.MouseButton1Click:Connect(function()
    state.AutoAttackNPC = not state.AutoAttackNPC
    autoAttackBtn.Text = (state.AutoAttackNPC and "[ ON ] " or "[ OFF ] ") .. "Ataque à distância NPCs (Blox Fruits)"
    if state.AutoAttackNPC then
        startAutoAttackNPC()
    else
        stopAutoAttackNPC()
    end
end)

------------------------------------------------------------------------
-- Minimizar/Abrir Hub
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

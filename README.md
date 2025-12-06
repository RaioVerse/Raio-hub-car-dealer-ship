--[[
    RaioVerse Hub - LocalScript Roblox
    Funções: AimBot, Fly, Velocidade, Exibir Hitbox, Minimizar HUB
    Coloquem esse script em StarterGui.
--]]

local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local RunService = game:GetService("RunService")
local CoreGui = game:GetService("CoreGui")

local localPlayer = Players.LocalPlayer

------------------------------------------------------------------
---------------------- GUI DEFINITIONS ---------------------------
------------------------------------------------------------------

local function createRoundFrame(parent, size, pos, color, transparency)
    local frame = Instance.new("Frame")
    frame.Size = size
    frame.Position = pos
    frame.Parent = parent
    frame.BackgroundColor3 = color
    frame.BackgroundTransparency = transparency or 0
    frame.BorderSizePixel = 0
    local uw = Instance.new("UICorner", frame)
    uw.CornerRadius = UDim.new(0, 18)
    return frame
end

-- HUB principal
local hubGui = Instance.new("ScreenGui")
hubGui.Name = "RaioVerseHub"
hubGui.Parent = CoreGui

local mainFrame = createRoundFrame(
    hubGui, UDim2.new(0,390,0,310), UDim2.new(0.5,-195,0.5,-155), Color3.fromRGB(25,28,48), 0.10)
mainFrame.Active, mainFrame.Draggable, mainFrame.Visible = true, true, true

-- Header e botão minimizar
local header = createRoundFrame(mainFrame, UDim2.new(1,0,0,45), UDim2.new(0,0,0,0), Color3.fromRGB(30,37,70), 0.07)
local title = Instance.new("TextLabel", header)
title.Size = UDim2.new(1, -50, 1, 0)
title.Position = UDim2.new(0, 15, 0, 0)
title.BackgroundTransparency = 1
title.Font = Enum.Font.GothamBold
title.Text = "RaioVerse Hub"
title.TextColor3 = Color3.fromRGB(195,211,255)
title.TextSize = 28
title.TextXAlignment = Enum.TextXAlignment.Left

local minBtn = Instance.new("TextButton", header)
minBtn.Size = UDim2.new(0,38,0,30)
minBtn.Position = UDim2.new(1,-45,0,8)
minBtn.BackgroundColor3 = Color3.fromRGB(52,68,143)
minBtn.AutoButtonColor = true
minBtn.Text = "_"
minBtn.Font = Enum.Font.GothamBold
minBtn.TextColor3 = Color3.fromRGB(205,205,230)
minBtn.TextSize = 28
Instance.new("UICorner", minBtn).CornerRadius = UDim.new(0, 8)

-- Container de botões
local btnFrame = createRoundFrame(mainFrame, UDim2.new(1, -32, 1, -60), UDim2.new(0,16,0,50), Color3.fromRGB(29,23,43), 0.18)
local btnList = Instance.new("UIListLayout", btnFrame)
btnList.Padding = UDim.new(0,16)
btnList.HorizontalAlignment = Enum.HorizontalAlignment.Center
btnList.SortOrder = Enum.SortOrder.LayoutOrder

-- Função para criar botões toggláveis
local function makeToggleBtn(texto)
    local btn = Instance.new("TextButton", btnFrame)
    btn.Size = UDim2.new(1, -10, 0, 44)
    btn.BackgroundColor3 = Color3.fromRGB(67, 89, 174)
    btn.AutoButtonColor = true
    btn.Text = "[ OFF ]  " .. texto
    btn.Font = Enum.Font.GothamSemibold
    btn.TextColor3 = Color3.fromRGB(235,235,255)
    btn.TextSize = 19
    local icorner = Instance.new("UICorner", btn)
    icorner.CornerRadius = UDim.new(0,13)
    -- Hover effect
    btn.MouseEnter:Connect(function() btn.BackgroundColor3 = Color3.fromRGB(88,115,215) end)
    btn.MouseLeave:Connect(function() btn.BackgroundColor3 = Color3.fromRGB(67,89,174) end)
    return btn
end

-- Créditos discretos
local credits = Instance.new("TextLabel", mainFrame)
credits.Size = UDim2.new(1,0,0,18)
credits.Position = UDim2.new(0,0,1,-18)
credits.BackgroundTransparency = 1
credits.Text = "☄️ By RaioVerse"
credits.TextColor3 = Color3.fromRGB(150, 190, 255)
credits.TextStrokeTransparency = 0.7
credits.Font = Enum.Font.GothamSemibold
credits.TextSize = 14

------------------------------------------------------------------
---------------------- TOGGLE LOGIC ------------------------------
------------------------------------------------------------------

-- Estados
local state = {
    AimBot = false,
    Fly = false,
    Speed = false,
    Hitbox = false
}

------------------------------------------------------------------
-- AimBot: Câmera mira automaticamente no jogador mais próximo da sua tela enquanto pressionando [Q] --
------------------------------------------------------------------
local AimbotAimKey = Enum.KeyCode.Q
local currentAimbot = nil

local function getClosestPlayer()
    local smallest = math.huge
    local cam = workspace.CurrentCamera
    local bestPlayer = nil
    for _,p in pairs(Players:GetPlayers()) do
        if p ~= localPlayer and p.Character and p.Character:FindFirstChild("HumanoidRootPart") and p.Character:FindFirstChild("Humanoid") and p.Character.Humanoid.Health > 0 then
            local hrp = p.Character.HumanoidRootPart.Position
            local pos, onScreen = cam:WorldToViewportPoint(hrp)
            if onScreen then
                local dist = (Vector2.new(pos.X, pos.Y) - UserInputService:GetMouseLocation()).Magnitude
                if dist < smallest then
                    smallest = dist
                    bestPlayer = p
                end
            end
        end
    end
    return bestPlayer
end

local function doAimbot()
    if not state.AimBot then return end
    local cam = workspace.CurrentCamera
    local target = getClosestPlayer()
    if target and target.Character and target.Character:FindFirstChild("HumanoidRootPart") then
        cam.CFrame = CFrame.new(cam.CFrame.Position, target.Character.HumanoidRootPart.Position)
    end
end

local aimBotConnection = nil

------------------------------------------------------------------
-- Fly: Segure Barra de Espaço para subir/se mover livremente
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
        flyT.Velocity = flyVec.Unit * FlySpeed
        if flyVec.Magnitude == 0 then flyT.Velocity = Vector3.new(0,0,0) end
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
-- Speed: aumenta/diminui a WalkSpeed do player
------------------------------------------------------------------
local SpeedValue = 45
local function setSpeed(enabled)
    if localPlayer.Character and localPlayer.Character:FindFirstChildWhichIsA("Humanoid") then
        localPlayer.Character:FindFirstChildWhichIsA("Humanoid").WalkSpeed = enabled and SpeedValue or 16
    end
end
local function FixSpeed()
    setSpeed(state.Speed)
end
localPlayer.CharacterAdded:Connect(function()
    wait(0.5)
    FixSpeed()
end)

------------------------------------------------------------------
-- Hitbox: destaca Hitbox de outros players
------------------------------------------------------------------
local hitboxParts = {}
local function highlightHitboxes(enable)
    -- Limpa os highlights antigos
    for _, hl in ipairs(hitboxParts) do pcall(function() hl:Destroy() end) end
    table.clear(hitboxParts)
    if not enable then return end
    for _, player in ipairs(Players:GetPlayers()) do
        if player ~= localPlayer and player.Character and player.Character:FindFirstChild("HumanoidRootPart") then
            local highlight = Instance.new("BoxHandleAdornment")
            highlight.Name = "RaioHitbox"
            highlight.Adornee = player.Character.HumanoidRootPart
            highlight.Color3 = Color3.fromRGB(255, 65, 85)
            highlight.AlwaysOnTop, highlight.Transparency, highlight.Size = true, 0.6, Vector3.new(4,6,2)
            highlight.ZIndex = 3
            highlight.Parent = workspace
            table.insert(hitboxParts, highlight)
        end
    end
end
Players.PlayerAdded:Connect(function(plr)
    plr.CharacterAdded:Connect(function()
        if state.Hitbox then wait(0.2) highlightHitboxes(true) end
    end)
end)
Players.PlayerRemoving:Connect(function(plr)
    if state.Hitbox then highlightHitboxes(true) end
end)

------------------------------------------------------------------
-- BOTÕES E EVENTOS
------------------------------------------------------------------
local aimBtn = makeToggleBtn("Aim Bot (Segure Q para mirar automático)")
aimBtn.MouseButton1Click:Connect(function()
    state.AimBot = not state.AimBot
    aimBtn.Text = (state.AimBot and "[ ON ] " or "[ OFF ] ") .. "Aim Bot (Segure Q para mirar automático)"
    if state.AimBot and not aimBotConnection then
        aimBotConnection = UserInputService.InputBegan:Connect(function(input,gp)
            if input.KeyCode == AimbotAimKey then
                RunService:BindToRenderStep("RaioVerseAim", 300, doAimbot)
            end
        end)
        UserInputService.InputEnded:Connect(function(input,gp)
            if input.KeyCode == AimbotAimKey then
                RunService:UnbindFromRenderStep("RaioVerseAim")
            end
        end)
    elseif not state.AimBot and aimBotConnection then
        aimBotConnection:Disconnect()
        aimBotConnection = nil
        RunService:UnbindFromRenderStep("RaioVerseAim")
    end
end)

local flyBtn = makeToggleBtn("Fly (Use W/A/S/D + Espaço/LCtrl)")
flyBtn.MouseButton1Click:Connect(function()
    state.Fly = not state.Fly
    flyBtn.Text = (state.Fly and "[ ON ] " or "[ OFF ] ") .. "Fly (Use W/A/S/D + Espaço/LCtrl)"
    if state.Fly then
        flyActivate()
    else
        stopFly()
    end
end)

local speedBtn = makeToggleBtn("Velocidade Turbo ("..SpeedValue..")")
speedBtn.MouseButton1Click:Connect(function()
    state.Speed = not state.Speed
    speedBtn.Text = (state.Speed and "[ ON ] " or "[ OFF ] ") .. "Velocidade Turbo ("..SpeedValue..")"
    setSpeed(state.Speed)
end)

local hitboxBtn = makeToggleBtn("Ativar Hitbox Jogadores")
hitboxBtn.MouseButton1Click:Connect(function()
    state.Hitbox = not state.Hitbox
    hitboxBtn.Text = (state.Hitbox and "[ ON ] " or "[ OFF ] ") .. "Ativar Hitbox Jogadores"
    highlightHitboxes(state.Hitbox)
end)

------------------------------------------------------------------
---------------------- MINIMIZAR/RESTAURAR HUB -------------------
------------------------------------------------------------------
local minimized = false
local origSize, origPos = mainFrame.Size, mainFrame.Position

minBtn.MouseButton1Click:Connect(function()
    minimized = not minimized
    if minimized then
        mainFrame:TweenSize(UDim2.new(0,190, 0,60), "Out", "Quad", 0.25, true)
        mainFrame:TweenPosition(UDim2.new(0, 30, 0, 30), "Out", "Quad", 0.22, true)
        btnFrame.Visible = false
        for _,c in pairs(mainFrame:GetChildren()) do if c:IsA("TextLabel") and c ~= header and c ~= title then c.Visible = false end end
        minBtn.Text = "□"
    else
        mainFrame:TweenSize(origSize, "Out", "Quad", 0.23, true)
        mainFrame:TweenPosition(origPos, "Out", "Quad", 0.22, true)
        btnFrame.Visible = true
        for _,c in pairs(mainFrame:GetChildren()) do if c:IsA("TextLabel") then c.Visible = true end end
        minBtn.Text = "_"
    end
end)

------------------------------------------------------------------
---- FEEDBACK VISUAL E FUNCIONALIDADES BASE CONCLUÍDAS -----------
---- Sinta-se à vontade para editar textos, cores, ícones! -------
------------------------------------------------------------------

-- GUI/Hub ficará sempre acima
hubGui.DisplayOrder = 8926

-- Garantir que WalkSpeed sempre ajuste ao respawn
game:GetService("RunService").Stepped:Connect(function()
    if state.Speed then setSpeed(true) end
end)

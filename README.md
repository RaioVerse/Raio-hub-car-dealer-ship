--[[
  Script Local: Hub Roblox bonito e funcional
  Opções: Aim Bot, Fly, Controlador de Velocidade, Farmador de Dinheiro (Car Dealership)
--]]

-- Gui Setup
local player = game.Players.LocalPlayer
local coreGui = game:GetService("CoreGui")

local hub = Instance.new("ScreenGui")
hub.Name = "RaioHub"
hub.Parent = coreGui

local mainFrame = Instance.new("Frame")
mainFrame.Parent = hub
mainFrame.Position = UDim2.new(0.5, -175, 0.5, -150)
mainFrame.Size = UDim2.new(0, 350, 0, 300)
mainFrame.BackgroundColor3 = Color3.fromRGB(35, 35, 50)
mainFrame.BorderSizePixel = 0
mainFrame.BackgroundTransparency = 0.12
mainFrame.Active = true
mainFrame.Draggable = true
mainFrame.AnchorPoint = Vector2.new(0.5, 0.5)
mainFrame.Name = "MainFrame"

local title = Instance.new("TextLabel")
title.Parent = mainFrame
title.Size = UDim2.new(1, 0, 0, 40)
title.Text = "RaioVerse Hub"
title.Font = Enum.Font.GothamBold
title.TextSize = 24
title.TextColor3 = Color3.fromRGB(255, 255, 255)
title.BackgroundTransparency = 1

local uiList = Instance.new("UIListLayout")
uiList.Parent = mainFrame
uiList.HorizontalAlignment = Enum.HorizontalAlignment.Center
uiList.SortOrder = Enum.SortOrder.LayoutOrder
uiList.Padding = UDim.new(0,16)

-- Helper to build buttons
local function createToggleBtn(name)
    local btn = Instance.new("TextButton")
    btn.Size = UDim2.new(0.8, 0, 0, 40)
    btn.Text = "[ OFF ]  " .. name
    btn.Font = Enum.Font.GothamSemibold
    btn.TextColor3 = Color3.fromRGB(220,220,220)
    btn.BackgroundColor3 = Color3.fromRGB(60, 60, 95)
    btn.BorderSizePixel = 0
    btn.Name = name.."Btn"
    btn.LayoutOrder = 1
    return btn
end

-- State table
local state = {
    AimBot = false,
    Fly = false,
    SpeedControl = false,
    AutoFarm = false
}

---------------------------
-- [ AimBot Toggle ] ------
---------------------------
local aimBotBtn = createToggleBtn("Aim Bot")
aimBotBtn.Parent = mainFrame
aimBotBtn.MouseButton1Click:Connect(function()
    state.AimBot = not state.AimBot
    aimBotBtn.Text = ((state.AimBot and "[ ON ] ") or "[ OFF ] ") .. "Aim Bot"
    -- Adicione aqui o código do AimBot
    if state.AimBot then
        -- Ativar AimBot
    else
        -- Desativar AimBot
    end
end)

---------------------------
-- [ Fly Toggle ] ---------
---------------------------
local flyBtn = createToggleBtn("Fly")
flyBtn.Parent = mainFrame
flyBtn.MouseButton1Click:Connect(function()
    state.Fly = not state.Fly
    flyBtn.Text = ((state.Fly and "[ ON ] ") or "[ OFF ] ") .. "Fly"
    -- Adicione aqui o código do Fly
    if state.Fly then
        -- Ativar Fly
    else
        -- Desativar Fly
    end
end)

---------------------------
-- [ Speed Control ] ------
---------------------------
local speedBtn = createToggleBtn("Controlador de Velocidade")
speedBtn.Parent = mainFrame
speedBtn.MouseButton1Click:Connect(function()
    state.SpeedControl = not state.SpeedControl
    speedBtn.Text = ((state.SpeedControl and "[ ON ] ") or "[ OFF ] ") .. "Controlador de Velocidade"
    if state.SpeedControl then
        player.Character.Humanoid.WalkSpeed = 50 -- exemplo: aumentar para 50
    else
        player.Character.Humanoid.WalkSpeed = 16 -- padrão do Roblox
    end
end)

---------------------------
-- [ AutoFarm Money ] -----
---------------------------
local farmBtn = createToggleBtn("Farmador de Dinheiro (Car Dealership)")
farmBtn.Parent = mainFrame
farmBtn.MouseButton1Click:Connect(function()
    state.AutoFarm = not state.AutoFarm
    farmBtn.Text = ((state.AutoFarm and "[ ON ] ") or "[ OFF ] ") .. "Farmador de Dinheiro (Car Dealership)"
    if state.AutoFarm then
        -- Farm ativado (coloque a lógica do farm aqui!)
        -- Exemplo: coroutine para farm automático
        state._farmThread = coroutine.create(function()
            while state.AutoFarm do
                -- Lógica customizada para seu jogo de Car Dealership aqui!
                -- Exemplo: disparar eventos, teleportar, vender carros etc
                wait(1)
            end
        end)
        coroutine.resume(state._farmThread)
    else
        -- Farm desativado
    end
end)

-- Créditos discretos
local credit = Instance.new("TextLabel")
credit.Parent = mainFrame
credit.Size = UDim2.new(1,0,0,20)
credit.Text = "Feito por RaioVerse"
credit.Font = Enum.Font.Gotham
credit.TextSize = 13
credit.TextColor3 = Color3.fromRGB(160,160,255)
credit.BackgroundTransparency = 1
credit.LayoutOrder = 99

---------------------------
-- GUI finalizada!
---------------------------
-- Basta colocar este script em StarterGui como LocalScript
-- E adicionar o código específico das funções conforme seu jogo :)

-- Para mais funcionalidades visuais, você pode explorar plugins de UI como Material Icons, TweenService para animações, etc!

-- ANTI-ATTACK GUI
-- LocalScript em StarterPlayerScripts
-- Requer ProtectionEvent e ProtectionServer no servidor

local Players = game:GetService("Players")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local UserInputService = game:GetService("UserInputService")

local player = Players.LocalPlayer
local event = ReplicatedStorage:WaitForChild("ProtectionEvent")

local protectionOn = true
local alertsOn = true

local gui = Instance.new("ScreenGui")
gui.Name = "AntiAttackGui"
gui.ResetOnSpawn = false
gui.Parent = player:WaitForChild("PlayerGui")

local panel = Instance.new("Frame")
panel.Size = UDim2.fromOffset(270, 240)
panel.Position = UDim2.new(0.5, -135, 0.5, -120)
panel.BackgroundColor3 = Color3.fromRGB(25, 29, 39)
panel.BorderSizePixel = 0
panel.Active = true
panel.Parent = gui

Instance.new("UICorner", panel).CornerRadius =
    UDim.new(0, 12)

local function label(text, y, height, color)
    local obj = Instance.new("TextLabel")
    obj.Size = UDim2.new(1, -20, 0, height)
    obj.Position = UDim2.fromOffset(10, y)
    obj.BackgroundTransparency = 1
    obj.Text = text
    obj.TextColor3 = color or Color3.new(1, 1, 1)
    obj.Font = Enum.Font.GothamBold
    obj.TextScaled = true
    obj.Parent = panel
    return obj
end

local function button(text, y, color)
    local obj = Instance.new("TextButton")
    obj.Size = UDim2.new(1, -20, 0, 38)
    obj.Position = UDim2.fromOffset(10, y)
    obj.BackgroundColor3 = color
    obj.TextColor3 = Color3.new(1, 1, 1)
    obj.Text = text
    obj.Font = Enum.Font.GothamBold
    obj.TextScaled = true
    obj.BorderSizePixel = 0
    obj.Parent = panel

    Instance.new("UICorner", obj).CornerRadius =
        UDim.new(0, 8)

    return obj
end

label("ANTI-ATTACK", 8, 28)

local status = label(
    "PROTEÇÃO: ATIVADA", 36, 23,
    Color3.fromRGB(80, 220, 130)
)

local toggle = button(
    "DESATIVAR PROTEÇÃO", 65,
    Color3.fromRGB(35, 145, 85)
)

local alerts = button(
    "DESATIVAR ALERTAS", 109,
    Color3.fromRGB(70, 90, 150)
)

local close = button(
    "OCULTAR PAINEL", 153,
    Color3.fromRGB(155, 55, 65)
)

local reopen = Instance.new("TextButton")
reopen.Name = "ReabrirAntiAttack"
reopen.Size = UDim2.fromOffset(145, 40)
reopen.Position = UDim2.new(0, 15, 0.5, 0)
reopen.BackgroundColor3 = Color3.fromRGB(35, 145, 85)
reopen.TextColor3 = Color3.new(1, 1, 1)
reopen.Text = "ABRIR ANTI-ATTACK"
reopen.Font = Enum.Font.GothamBold
reopen.TextScaled = true
reopen.Visible = false
reopen.Parent = gui

Instance.new("UICorner", reopen).CornerRadius =
    UDim.new(0, 8)

toggle.Activated:Connect(function()
    protectionOn = not protectionOn

    toggle.Text = protectionOn
        and "DESATIVAR PROTEÇÃO"
        or "ATIVAR PROTEÇÃO"

    toggle.BackgroundColor3 = protectionOn
        and Color3.fromRGB(35, 145, 85)
        or Color3.fromRGB(155, 55, 65)

    status.Text = protectionOn
        and "PROTEÇÃO: ATIVADA"
        or "PROTEÇÃO: DESATIVADA"

    status.TextColor3 = protectionOn
        and Color3.fromRGB(80, 220, 130)
        or Color3.fromRGB(255, 100, 100)

    event:FireServer("ToggleProtection", protectionOn)
end)

alerts.Activated:Connect(function()
    alertsOn = not alertsOn

    alerts.Text = alertsOn
        and "DESATIVAR ALERTAS"
        or "ATIVAR ALERTAS"

    alerts.BackgroundColor3 = alertsOn
        and Color3.fromRGB(70, 90, 150)
        or Color3.fromRGB(90, 90, 90)
end)

close.Activated:Connect(function()
    panel.Visible = false
    reopen.Visible = true
end)

reopen.Activated:Connect(function()
    panel.Visible = true
    reopen.Visible = false
end)

event.OnClientEvent:Connect(function(action, state)
    if action == "ProtectionStatus" then
        status.Text = state
            and "PROTEÇÃO: ATIVADA"
            or "PROTEÇÃO: DESATIVADA"
    elseif action == "SuspiciousAttack" and alertsOn then
        status.Text = "ALERTA: ATAQUE SUSPEITO"
        status.TextColor3 = Color3.fromRGB(255, 90, 70)
    end
end)

-- Arrastar painel com mouse ou toque
local dragging = false
local dragStart
local startPosition
local dragInput

panel.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1
        or input.UserInputType == Enum.UserInputType.Touch then

        dragging = true
        dragStart = input.Position
        startPosition = panel.Position
        dragInput = input
    end
end)

UserInputService.InputChanged:Connect(function(input)
    if dragging and (
        input.UserInputType == Enum.UserInputType.MouseMovement
        or input.UserInputType == Enum.UserInputType.Touch
    ) then
        local delta = input.Position - dragStart

        panel.Position = UDim2.new(
            startPosition.X.Scale,
            startPosition.X.Offset + delta.X,
            startPosition.Y.Scale,
            startPosition.Y.Offset + delta.Y
        )
    end
end)

UserInputService.InputEnded:Connect(function(input)
    if input == dragInput then
        dragging = false
        dragInput = nil
    end
end)

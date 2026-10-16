local Players = game:GetService("Players")

local player = Players.LocalPlayer
local enabled = false
local RANGE = 10000000
local COOLDOWN = 0
local lastAttack = 0

-- GUI
local gui = Instance.new("ScreenGui")
gui.Name = "InstantKillGui"
gui.ResetOnSpawn = false
gui.Parent = player:WaitForChild("PlayerGui")

-- Painel principal
local panel = Instance.new("Frame")
panel.Size = UDim2.fromOffset(220, 150)
panel.Position = UDim2.new(0.5, -110, 0.5, -75)
panel.BackgroundColor3 = Color3.fromRGB(30, 30, 35)
panel.BorderSizePixel = 0
panel.Parent = gui

local panelCorner = Instance.new("UICorner")
panelCorner.CornerRadius = UDim.new(0, 12)
panelCorner.Parent = panel

-- Título
local title = Instance.new("TextLabel")
title.Size = UDim2.new(1, 0, 0, 30)
title.Position = UDim2.fromOffset(0, 5)
title.BackgroundTransparency = 1
title.Text = "INSTANT KILL"
title.TextColor3 = Color3.new(1, 1, 1)
title.TextScaled = true
title.Font = Enum.Font.GothamBold
title.Parent = panel

-- Botão ON/OFF
local toggle = Instance.new("TextButton")
toggle.Size = UDim2.new(1, -20, 0, 45)
toggle.Position = UDim2.fromOffset(10, 40)
toggle.Text = "INSTANT KILL: OFF"
toggle.TextScaled = true
toggle.Font = Enum.Font.GothamBold
toggle.BackgroundColor3 = Color3.fromRGB(170, 40, 40)
toggle.TextColor3 = Color3.new(1, 1, 1)
toggle.BorderSizePixel = 0
toggle.Parent = panel

local toggleCorner = Instance.new("UICorner")
toggleCorner.CornerRadius = UDim.new(0, 8)
toggleCorner.Parent = toggle

-- Botão atacar
local attack = Instance.new("TextButton")
attack.Size = UDim2.new(1, -20, 0, 45)
attack.Position = UDim2.fromOffset(10, 90)
attack.Text = "ATACAR"
attack.TextScaled = true
attack.Font = Enum.Font.GothamBold
attack.BackgroundColor3 = Color3.fromRGB(50, 100, 200)
attack.TextColor3 = Color3.new(1, 1, 1)
attack.BorderSizePixel = 0
attack.Parent = panel

local attackCorner = Instance.new("UICorner")
attackCorner.CornerRadius = UDim.new(0, 8)
attackCorner.Parent = attack

-- Texto "by Caos"
local credit = Instance.new("TextLabel")
credit.Size = UDim2.fromOffset(220, 25)
credit.Position = UDim2.new(0, 0, 1, 5)
credit.BackgroundTransparency = 1
credit.Text = "by Caos"
credit.TextColor3 = Color3.fromRGB(255, 0, 0)
credit.TextScaled = true
credit.Font = Enum.Font.Gotham
credit.Parent = panel

-- Ativar/desativar
toggle.MouseButton1Click:Connect(function()
	enabled = not enabled

	if enabled then
		toggle.Text = "INSTANT KILL: ON"
		toggle.BackgroundColor3 = Color3.fromRGB(40, 170, 70)
	else
		toggle.Text = "INSTANT KILL: OFF"
		toggle.BackgroundColor3 = Color3.fromRGB(170, 40, 40)
	end
end)

-- Ataque
attack.MouseButton1Click:Connect(function()
	if not enabled then return end

	local now = os.clock()
	if now - lastAttack < COOLDOWN then return end
	lastAttack = now

	local character = player.Character
	if not character then return end

	local root = character:FindFirstChild("HumanoidRootPart")
	if not root then return end

	for _, target in ipairs(Players:GetPlayers()) do
		if target ~= player then
			local targetCharacter = target.Character
			local humanoid = targetCharacter
				and targetCharacter:FindFirstChildOfClass("Humanoid")
			local targetRoot = targetCharacter
				and targetCharacter:FindFirstChild("HumanoidRootPart")

			if humanoid and targetRoot and humanoid.Health > 0 then
				local distance =
					(targetRoot.Position - root.Position).Magnitude

				if distance <= RANGE then
					humanoid.Health = 0
				end
			end
		end
	end
end)

-- =====================================================
-- SISTEMA PARA ARRASTAR O PAINEL
-- =====================================================

local UserInputService = game:GetService("UserInputService")

local dragging = false
local dragStart
local startPosition

local function updateDrag(input)
	local delta = input.Position - dragStart

	panel.Position = UDim2.new(
		startPosition.X.Scale,
		startPosition.X.Offset + delta.X,
		startPosition.Y.Scale,
		startPosition.Y.Offset + delta.Y
	)
end

panel.InputBegan:Connect(function(input)
	if input.UserInputType == Enum.UserInputType.MouseButton1
		or input.UserInputType == Enum.UserInputType.Touch then

		dragging = true
		dragStart = input.Position
		startPosition = panel.Position

		input.Changed:Connect(function()
			if input.UserInputState == Enum.UserInputState.End then
				dragging = false
			end
		end)
	end
end)

UserInputService.InputChanged:Connect(function(input)
	if dragging and (
		input.UserInputType == Enum.UserInputType.MouseMovement
		or input.UserInputType == Enum.UserInputType.Touch
	) then
		updateDrag(input)
	end
end)

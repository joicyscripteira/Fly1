-- Fly Script com botão para celular - por @joicyscripteira (modificado)

local UIS = game:GetService("UserInputService")
local RunService = game:GetService("RunService")
local Players = game:GetService("Players")

local player = Players.LocalPlayer
local character = player.Character or player.CharacterAdded:Wait()
local humanoidRootPart = character:WaitForChild("HumanoidRootPart")

local flying = false
local speed = 50
local control = {F = 0, B = 0, L = 0, R = 0}
local flyConnection = nil

-- Função para voar
local function fly()
	if flying then return end
	flying = true

	local bodyGyro = Instance.new("BodyGyro", humanoidRootPart)
	bodyGyro.P = 9e4
	bodyGyro.MaxTorque = Vector3.new(9e9, 9e9, 9e9)
	bodyGyro.CFrame = humanoidRootPart.CFrame

	local bodyVelocity = Instance.new("BodyVelocity", humanoidRootPart)
	bodyVelocity.Velocity = Vector3.new(0, 0, 0)
	bodyVelocity.MaxForce = Vector3.new(9e9, 9e9, 9e9)

	flyConnection = RunService.RenderStepped:Connect(function()
		if not flying then return end
		bodyGyro.CFrame = workspace.CurrentCamera.CFrame

		local move = Vector3.new(control.L + control.R, 0, control.F + control.B)
		if move.Magnitude > 0 then
			bodyVelocity.Velocity = (workspace.CurrentCamera.CFrame:VectorToWorldSpace(move.Unit)) * speed
		else
			bodyVelocity.Velocity = Vector3.zero
		end
	end)

	-- Parar quando morrer
	player.CharacterRemoving:Connect(function()
		flying = false
		bodyGyro:Destroy()
		bodyVelocity:Destroy()
	end)
end

-- Função para parar de voar
local function stopFlying()
	flying = false
	if flyConnection then flyConnection:Disconnect() end
	if humanoidRootPart:FindFirstChild("BodyGyro") then
		humanoidRootPart.BodyGyro:Destroy()
	end
	if humanoidRootPart:FindFirstChild("BodyVelocity") then
		humanoidRootPart.BodyVelocity:Destroy()
	end
end

-- Criar botão na lateral direita
local screenGui = Instance.new("ScreenGui")
screenGui.Name = "FlyUI"
screenGui.ResetOnSpawn = false
screenGui.Parent = player:WaitForChild("PlayerGui")

local toggleButton = Instance.new("TextButton")
toggleButton.Size = UDim2.new(0, 100, 0, 40)
toggleButton.Position = UDim2.new(1, -110, 0.85, 0) -- Lateral direita inferior
toggleButton.AnchorPoint = Vector2.new(0, 0)
toggleButton.BackgroundColor3 = Color3.fromRGB(200, 30, 30)
toggleButton.Text = "Fly: OFF"
toggleButton.TextScaled = true
toggleButton.Font = Enum.Font.SourceSansBold
toggleButton.TextColor3 = Color3.new(1, 1, 1)
toggleButton.Parent = screenGui

toggleButton.MouseButton1Click:Connect(function()
	if flying then
		stopFlying()
		toggleButton.Text = "Fly: OFF"
		toggleButton.BackgroundColor3 = Color3.fromRGB(200, 30, 30)
	else
		fly()
		toggleButton.Text = "Fly: ON"
		toggleButton.BackgroundColor3 = Color3.fromRGB(30, 200, 30)
	end
end)

-- Suporte a controles WASD no PC
UIS.InputBegan:Connect(function(input)
	if input.KeyCode == Enum.KeyCode.W then control.F = -1 end
	if input.KeyCode == Enum.KeyCode.S then control.B = 1 end
	if input.KeyCode == Enum.KeyCode.A then control.L = -1 end
	if input.KeyCode == Enum.KeyCode.D then control.R = 1 end
end)

UIS.InputEnded:Connect(function(input)
	if input.KeyCode == Enum.KeyCode.W then control.F = 0 end
	if input.KeyCode == Enum.KeyCode.S then control.B = 0 end
	if input.KeyCode == Enum.KeyCode.A then control.L = 0 end
	if input.KeyCode == Enum.KeyCode.D then control.R = 0 end
end)

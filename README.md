-- Script de Fly Roblox
-- Comandos:
-- ;fly = ativar voo
-- ;unfly = desativar voo

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UIS = game:GetService("UserInputService")

local player = Players.LocalPlayer
local flying = false
local speed = 50 -- Velocidade do voo

local bodyVelocity
local bodyGyro

-- Função para começar a voar
local function startFlying()
	if flying then return end
	flying = true

	local character = player.Character or player.CharacterAdded:Wait()
	local humanoidRootPart = character:WaitForChild("HumanoidRootPart")

	-- Mantém o jogador flutuando
	bodyVelocity = Instance.new("BodyVelocity")
	bodyVelocity.MaxForce = Vector3.new(4000, 4000, 4000)
	bodyVelocity.Velocity = Vector3.new(0, 0, 0)
	bodyVelocity.Parent = humanoidRootPart

	-- Mantém a rotação alinhada com a câmera
	bodyGyro = Instance.new("BodyGyro")
	bodyGyro.MaxTorque = Vector3.new(4000, 4000, 4000)
	bodyGyro.P = 3000
	bodyGyro.Parent = humanoidRootPart

	-- Faz o jogador parar de cair instantaneamente
	local humanoid = character:FindFirstChildOfClass("Humanoid")
	if humanoid then
		humanoid.PlatformStand = true
	end

	print("Fly ativado!")
end

-- Função para parar de voar
local function stopFlying()
	if not flying then return end
	flying = false

	if bodyVelocity then bodyVelocity:Destroy() end
	if bodyGyro then bodyGyro:Destroy() end

	local character = player.Character
	if character then
		local humanoid = character:FindFirstChildOfClass("Humanoid")
		if humanoid then
			humanoid.PlatformStand = false
		end
	end

	print("Fly desativado!")
end

-- Detecta mensagens no chat
player.Chatted:Connect(function(msg)
	msg = msg:lower()
	if msg == ";fly" then
		startFlying()
	elseif msg == ";unfly" then
		stopFlying()
	end
end)

-- Controle do movimento durante o voo
RunService.RenderStepped:Connect(function()
	if flying then
		local character = player.Character
		if not character or not character:FindFirstChild("HumanoidRootPart") then return end
		local hrp = character.HumanoidRootPart
		local camera = workspace.CurrentCamera

		local moveDirection = Vector3.new(0, 0, 0)

		-- Controles básicos
		if UIS:IsKeyDown(Enum.KeyCode.W) then
			moveDirection = moveDirection + camera.CFrame.LookVector
		end
		if UIS:IsKeyDown(Enum.KeyCode.S) then
			moveDirection = moveDirection - camera.CFrame.LookVector
		end
		if UIS:IsKeyDown(Enum.KeyCode.A) then
			moveDirection = moveDirection - camera.CFrame.RightVector
		end
		if UIS:IsKeyDown(Enum.KeyCode.D) then
			moveDirection = moveDirection + camera.CFrame.RightVector
		end
		if UIS:IsKeyDown(Enum.KeyCode.Space) then
			moveDirection = moveDirection + Vector3.new(0, 1, 0)
		end
		if UIS:IsKeyDown(Enum.KeyCode.LeftShift) then
			moveDirection = moveDirection - Vector3.new(0, 1, 0)
		end

		-- Se não estiver apertando nada, ele fica parado no ar
		if moveDirection.Magnitude > 0 then
			bodyVelocity.Velocity = moveDirection.Unit * speed
		else
			bodyVelocity.Velocity = Vector3.new(0, 0, 0)
		end

		bodyGyro.CFrame = camera.CFrame
	end
end)

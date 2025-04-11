local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local LocalPlayer = Players.LocalPlayer

-- Criar GUI
local ScreenGui = Instance.new("ScreenGui", game.CoreGui)
ScreenGui.ResetOnSpawn = false

local Frame = Instance.new("Frame")
Frame.Size = UDim2.new(0, 180, 0, 200)
Frame.Position = UDim2.new(0, 30, 0.5, -100)
Frame.BackgroundColor3 = Color3.fromRGB(25, 25, 25)
Frame.BorderSizePixel = 0
Frame.Active = true
Frame.Parent = ScreenGui

-- Deixar movível no mobile/PC
local dragging, dragInput, dragStart, startPos

local function updateInput(input)
	local delta = input.Position - dragStart
	Frame.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
end

Frame.InputBegan:Connect(function(input)
	if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
		dragging = true
		dragStart = input.Position
		startPos = Frame.Position

		input.Changed:Connect(function()
			if input.UserInputState == Enum.UserInputState.End then
				dragging = false
			end
		end)
	end
end)

Frame.InputChanged:Connect(function(input)
	if input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch then
		dragInput = input
	end
end)

UserInputService.InputChanged:Connect(function(input)
	if input == dragInput and dragging then
		updateInput(input)
	end
end)

-- Criar botão
local function criarBotao(texto, aoClicar)
	local botao = Instance.new("TextButton", Frame)
	botao.Size = UDim2.new(1, -10, 0, 40)
	botao.Position = UDim2.new(0, 5, 0, #Frame:GetChildren() * 45 - 30)
	botao.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
	botao.TextColor3 = Color3.fromRGB(255, 255, 255)
	botao.Font = Enum.Font.SourceSansBold
	botao.TextSize = 16
	botao.BorderSizePixel = 0
	botao.Text = texto

	botao.MouseButton1Click:Connect(aoClicar)
end

-- Função para puxar tool por nome (ou similar)
local function puxarToolPorNome(nomesPossiveis)
	for _, localBusca in ipairs({workspace, game:GetService("ReplicatedStorage")}) do
		for _, tool in ipairs(localBusca:GetDescendants()) do
			if tool:IsA("Tool") then
				for _, nome in ipairs(nomesPossiveis) do
					if string.lower(tool.Name):find(string.lower(nome)) then
						local sucesso, erro = pcall(function()
							local clone = tool:Clone()
							clone.Parent = nil -- inventário nil
						end)
						if not sucesso then
							warn("Erro ao puxar tool:", erro)
						end
						return
					end
				end
			end
		end
	end
end

-- Função para remover todas as tools do inventário nil (desequipar)
local function removerToolsNil()
	for _, obj in ipairs(LocalPlayer:GetDescendants()) do
		if obj:IsA("Tool") then
			obj.Parent = nil
		end
	end
end

-- Criar botões
criarBotao("Puxar AR-15", function()
	puxarToolPorNome({"AR-15", "Ar-15"})
end)

criarBotao("Puxar Uzi", function()
	puxarToolPorNome({"Uzi", "UZI"})
end)

criarBotao("Puxar Parafal", function()
	puxarToolPorNome({"PARAFAL", "Parafal", "Para", "fal"})
end)

criarBotao("Tirar todas as Tools", function()
	removerToolsNil()
end)

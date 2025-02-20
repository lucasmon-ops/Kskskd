local players = game:GetService("Players")
local localPlayer = players.LocalPlayer
local camera = game.Workspace.CurrentCamera
local userInput = game:GetService("UserInputService")
local starterGui = game:GetService("StarterGui")

local espEnabled = false
local aimbotEnabled = false
local highlights = {}

-- Criar interface gráfica (GUI)
local screenGui = Instance.new("ScreenGui")
screenGui.Parent = game.CoreGui

local function createButton(name, position, callback)
    local button = Instance.new("TextButton")
    button.Parent = screenGui
    button.Size = UDim2.new(0.2, 0, 0.1, 0)
    button.Position = position
    button.Text = name
    button.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
    button.TextColor3 = Color3.fromRGB(255, 255, 255)
    button.Font = Enum.Font.SourceSansBold
    button.TextScaled = true
    button.MouseButton1Click:Connect(callback)
end

-- Função para adicionar ESP (Highlight) nos inimigos
local function toggleESP()
    espEnabled = not espEnabled
    if not espEnabled then
        for _, highlight in pairs(highlights) do
            highlight:Destroy()
        end
        highlights = {}
        return
    end
    
    for _, player in pairs(players:GetPlayers()) do
        if player ~= localPlayer and player.Character and not highlights[player] then
            local highlight = Instance.new("Highlight")
            highlight.Parent = player.Character
            highlight.FillColor = Color3.fromRGB(255, 0, 0) -- Vermelho para inimigos
            highlight.OutlineColor = Color3.fromRGB(255, 255, 255) -- Contorno branco
            highlight.DepthMode = Enum.HighlightDepthMode.AlwaysOnTop
            highlights[player] = highlight
        end
    end
end

-- Atualiza ESP para novos jogadores
players.PlayerAdded:Connect(function(player)
    player.CharacterAdded:Connect(function()
        if espEnabled then
            toggleESP()
        end
    end)
end)

-- Função para encontrar o inimigo mais próximo
local function getClosestEnemy()
    local closest = nil
    local shortestDistance = math.huge

    for _, player in pairs(players:GetPlayers()) do
        if player ~= localPlayer and player.Character and player.Character:FindFirstChild("Head") then
            local headPosition = camera:WorldToViewportPoint(player.Character.Head.Position)
            local distance = (Vector2.new(camera.ViewportSize.X / 2, camera.ViewportSize.Y / 2) - Vector2.new(headPosition.X, headPosition.Y)).Magnitude

            if distance < shortestDistance then
                shortestDistance = distance
                closest = player
            end
        end
    end
    return closest
end

-- Função para ativar/desativar Aimbot
local function toggleAimbot()
    aimbotEnabled = not aimbotEnabled
end

-- Aimbot quando botão for pressionado
local function activateAimbot()
    if aimbotEnabled then
        local target = getClosestEnemy()
        if target and target.Character and target.Character:FindFirstChild("Head") then
            camera.CFrame = CFrame.new(camera.CFrame.Position, target.Character.Head.Position)
        end
    end
end

-- Criar botões na tela para ativar/desativar ESP e Aimbot
createButton("ESP ON/OFF", UDim2.new(0.1, 0, 0.8, 0), toggleESP)
createButton("AIMBOT ON/OFF", UDim2.new(0.7, 0, 0.8, 0), toggleAimbot)
createButton("AIMBOT", UDim2.new(0.4, 0, 0.8, 0), activateAimbot)

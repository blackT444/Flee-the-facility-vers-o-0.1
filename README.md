-- Criar interface gráfica
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Parent = game.Players.LocalPlayer:WaitForChild("PlayerGui")
ScreenGui.Name = "ESPButtonGui"

-- Criar botão principal 🎮
local button = Instance.new("TextButton")
button.Parent = ScreenGui
button.Size = UDim2.new(0, 100, 0, 50)
button.Position = UDim2.new(0.5, -50, 0.5, -25)
button.Text = ""
button.BackgroundTransparency = 1

local icon = Instance.new("TextLabel")
icon.Parent = button
icon.Size = UDim2.new(1, 0, 1, 0)
icon.Text = "🎮"
icon.BackgroundTransparency = 1
icon.TextSize = 30

-- Criar painel de controle ESP e Speed
local Frame = Instance.new("Frame")
Frame.Size = UDim2.new(0.15, 0, 0.07, 0)
Frame.Position = UDim2.new(0.4, 0, 0.85, 0)
Frame.BackgroundTransparency = 0.5
Frame.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
Frame.Parent = ScreenGui

local ToggleButton = Instance.new("TextButton")
ToggleButton.Size = UDim2.new(0.45, 0, 1, 0)
ToggleButton.BackgroundTransparency = 1
ToggleButton.TextScaled = true
ToggleButton.Text = "ON"
ToggleButton.Parent = Frame

local SpeedButton = Instance.new("TextButton")
SpeedButton.Size = UDim2.new(0.45, 0, 1, 0)
SpeedButton.Position = UDim2.new(0.55, 0, 0, 0)
SpeedButton.BackgroundTransparency = 1
SpeedButton.TextScaled = true
SpeedButton.Text = "⚡"
SpeedButton.Parent = Frame

-- Permitir arrastar 🎮 e ⚡ com toque
local UserInputService = game:GetService("UserInputService")
local dragging, dragStart, startPos

local function enableDragging(object)
    object.InputBegan:Connect(function(input)
        -- Verifique se o tipo de entrada é Toque ou Clique do Mouse
        if input.UserInputType == Enum.UserInputType.Touch or input.UserInputType == Enum.UserInputType.MouseButton1 then
            dragging = true
            dragStart = input.Position
            startPos = object.Position
        end
    end)

    object.InputChanged:Connect(function(input)
        if dragging and (input.UserInputType == Enum.UserInputType.Touch or input.UserInputType == Enum.UserInputType.MouseMovement) then
            local delta = input.Position - dragStart
            object.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
        end
    end)

    object.InputEnded:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.Touch or input.UserInputType == Enum.UserInputType.MouseButton1 then
            dragging = false
        end
    end)
end

enableDragging(button)
enableDragging(SpeedButton)

-- Criar ESP dos computadores
local function GetSizeOfObject(part)
    if part:IsA("BasePart") then
        return part.Size
    end
    return Vector3.new(0, 0, 0)
end

local function RemovePreviousESP()
    local ESPFolder = workspace:FindFirstChild("ESPComputer")
    if ESPFolder then
        ESPFolder:Destroy()
    end
end

local function CreateNewESP()
    RemovePreviousESP()
    local ESPPC = Instance.new("Folder", workspace)
    ESPPC.Name = "ESPComputer"

    local map = workspace:FindFirstChild(tostring(game.ReplicatedStorage.CurrentMap.Value))
    if map then
        for _, child in ipairs(map:GetChildren()) do
            if child.Name == "ComputerTable" then
                local screen = child:FindFirstChild("Screen")
                if screen then
                    local Box = Instance.new("BoxHandleAdornment")
                    Box.Size = GetSizeOfObject(screen) + Vector3.new(0.5, 0.5, 0.5)
                    Box.Adornee = screen
                    Box.AlwaysOnTop = true
                    Box.ZIndex = 5
                    Box.Transparency = 0.2
                    Box.Color3 = screen.Color
                    Box.Parent = ESPPC
                end
            end
        end
    end
end

button.MouseButton1Click:Connect(CreateNewESP)

-- Criar ESP nos jogadores
local showESP = true

local function createESP(player)
    if player.Character and player.Character:FindFirstChild("Head") then
        local highlight = Instance.new("Highlight")
        highlight.Name = "ESP_Highlight"
        highlight.Parent = player.Character
        highlight.FillTransparency = 1
        highlight.OutlineColor = Color3.fromRGB(255, 255, 255)
        highlight.OutlineTransparency = 0
        highlight.Enabled = showESP
    end
end

local function addESPToPlayers()
    for _, player in pairs(game.Players:GetPlayers()) do
        if player ~= game.Players.LocalPlayer then
            createESP(player)
        end
    end
end

game.Players.PlayerAdded:Connect(function(player)
    player.CharacterAdded:Connect(function()
        wait(1)
        createESP(player)
    end)
end)

addESPToPlayers()

local function updateESP()
    for _, player in pairs(game.Players:GetPlayers()) do
        if player ~= game.Players.LocalPlayer and player.Character then
            local highlight = player.Character:FindFirstChild("ESP_Highlight")
            if highlight then
                highlight.Enabled = showESP
            end
        end
    end
end

local function toggleESP()
    showESP = not showESP
    updateESP()
    ToggleButton.Text = showESP and "ON" or "OFF"
end

ToggleButton.MouseButton1Click:Connect(toggleESP)

-- Ajuste de iluminação
local function adjustLighting()
    local lighting = game.Lighting
    lighting.Brightness = 1.5
    lighting.Ambient = Color3.fromRGB(100, 100, 100)
    lighting.OutdoorAmbient = Color3.fromRGB(180, 180, 180)
end

game:GetService("RunService").RenderStepped:Connect(adjustLighting)

-- Controle de velocidade
local player = game.Players.LocalPlayer
local walkSpeedNormal = 16
local walkSpeedFast = 25
local speedActive = false

local function toggleSpeed()
    speedActive = not speedActive
    local character = player.Character or player.CharacterAdded:Wait()
    local humanoid = character:FindFirstChild("Humanoid")

    if humanoid then
        humanoid.WalkSpeed = speedActive and walkSpeedFast or walkSpeedNormal
        SpeedButton.Text = speedActive and "⚡" or "⚡"
    end
end

SpeedButton.MouseButton1Click:Connect(toggleSpeed)

player.CharacterAdded:Connect(function(character)
    wait(1)
    local humanoid = character:FindFirstChild("Humanoid")
    if humanoid then
        humanoid.WalkSpeed = speedActive and walkSpeedFast or walkSpeedNormal
    end
end)

local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local CoreGui = game:GetService("CoreGui")
local RunService = game:GetService("RunService")
local Teams = game:GetService("Teams")

local LocalPlayer = Players.LocalPlayer

if not LocalPlayer then
    warn("LocalPlayer not found. UI script cannot run.")
    return
end

-- Create ScreenGui
local espScreenGui = Instance.new("ScreenGui")
espScreenGui.Name = "ESPScreenGui"
espScreenGui.Parent = CoreGui

-- Create Main Frame
local mainFrame = Instance.new("Frame")
mainFrame.Name = "MainFrame"
mainFrame.Size = UDim2.new(0, 250, 0, 350) -- Increased height for better spacing
mainFrame.Position = UDim2.new(0.5, -125, 0.5, -175) -- Center of screen
mainFrame.BackgroundColor3 = Color3.fromRGB(35, 35, 35) -- Darker background
mainFrame.BorderSizePixel = 0
mainFrame.Parent = espScreenGui

-- Add a UI Corner for rounded edges
local uiCorner = Instance.new("UICorner")
uiCorner.CornerRadius = UDim.new(0, 8)
uiCorner.Parent = mainFrame

-- Add a Title Bar
local titleBar = Instance.new("TextLabel")
titleBar.Name = "TitleBar"
titleBar.Size = UDim2.new(1, 0, 0, 30)
titleBar.Position = UDim2.new(0, 0, 0, 0)
titleBar.BackgroundColor3 = Color3.fromRGB(50, 50, 50) -- Slightly lighter title bar
titleBar.TextColor3 = Color3.fromRGB(255, 255, 255)
titleBar.Text = "ESP & Aimbot"
titleBar.Font = Enum.Font.SourceSansBold
titleBar.TextScaled = true
titleBar.TextXAlignment = Enum.TextXAlignment.Center
titleBar.TextYAlignment = Enum.TextYAlignment.Center
titleBar.Parent = mainFrame

-- Add Close Button
local closeButton = Instance.new("TextButton")
closeButton.Name = "CloseButton"
closeButton.Size = UDim2.new(0, 30, 0, 30)
closeButton.Position = UDim2.new(1, -30, 0, 0)
closeButton.BackgroundColor3 = Color3.fromRGB(200, 50, 50) -- Red color for close
closeButton.TextColor3 = Color3.fromRGB(255, 255, 255)
closeButton.Text = "X"
closeButton.Font = Enum.Font.SourceSansBold
closeButton.TextScaled = true
closeButton.Parent = titleBar

closeButton.MouseButton1Click:Connect(function()
    espScreenGui.Enabled = false -- Hide the UI
    -- Optionally, destroy the UI elements to free up memory
    -- espScreenGui:Destroy()
end)

-- Draggable logic for mainFrame
local dragging = false
local dragStart = Vector2.new(0, 0)
local frameStartPos = UDim2.new(0, 0, 0, 0)

titleBar.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
        dragging = true
        dragStart = input.Position
        frameStartPos = mainFrame.Position
    end
end)

UserInputService.InputChanged:Connect(function(input)
    if (input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch) and dragging then
        local delta = input.Position - dragStart
        mainFrame.Position = UDim2.new(frameStartPos.X.Scale, frameStartPos.X.Offset + delta.X, frameStartPos.Y.Scale, frameStartPos.Y.Offset + delta.Y)
    end
end)

UserInputService.InputEnded:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
        dragging = false
    end
end)

-- Placeholder for ESP Toggle Button
local espToggleButton = Instance.new("TextButton")
espToggleButton.Name = "ESPToggleButton"
espToggleButton.Size = UDim2.new(0.45, 0, 0, 30)
espToggleButton.Position = UDim2.new(0.05, 0, 0, 40)
espToggleButton.BackgroundColor3 = Color3.fromRGB(60, 60, 60)
espToggleButton.TextColor3 = Color3.fromRGB(255, 255, 255)
espToggleButton.Text = "Toggle ESP: ON"
espToggleButton.Font = Enum.Font.SourceSansBold
espToggleButton.TextScaled = true
espToggleButton.Parent = mainFrame

-- Placeholder for Aimbot Toggle Button
local aimbotToggleButton = Instance.new("TextButton")
aimbotToggleButton.Name = "AimbotToggleButton"
aimbotToggleButton.Size = UDim2.new(0.45, 0, 0, 30)
aimbotToggleButton.Position = UDim2.new(0.5, 0, 0, 40)
aimbotToggleButton.BackgroundColor3 = Color3.fromRGB(60, 60, 60)
aimbotToggleButton.TextColor3 = Color3.fromRGB(255, 255, 255)
aimbotToggleButton.Text = "Toggle Aimbot: OFF"
aimbotToggleButton.Font = Enum.Font.SourceSansBold
aimbotToggleButton.TextScaled = true
aimbotToggleButton.Parent = mainFrame

-- Placeholder for ESP Options Frame
local espOptionsFrame = Instance.new("Frame")
espOptionsFrame.Name = "ESPOptionsFrame"
espOptionsFrame.Size = UDim2.new(0.9, 0, 0, 100) -- Fixed height
espOptionsFrame.Position = UDim2.new(0.05, 0, 0, 80)
espOptionsFrame.BackgroundColor3 = Color3.fromRGB(45, 45, 45)
espOptionsFrame.BorderSizePixel = 0
espOptionsFrame.Parent = mainFrame

-- Placeholder for Aimbot Options Frame
local aimbotOptionsFrame = Instance.new("Frame")
aimbotOptionsFrame.Name = "AimbotOptionsFrame"
aimbotOptionsFrame.Size = UDim2.new(0.9, 0, 0, 70) -- Fixed height
aimbotOptionsFrame.Position = UDim2.new(0.05, 0, 0, 200)
aimbotOptionsFrame.BackgroundColor3 = Color3.fromRGB(45, 45, 45)
aimbotOptionsFrame.BorderSizePixel = 0
aimbotOptionsFrame.Parent = mainFrame

print("UI structure created.")


-- Aimbot and ESP Core Logic (Integrated)

local Character = LocalPlayer.Character or LocalPlayer.CharacterAdded:Wait()
local Humanoid = Character:WaitForChild("Humanoid")
local RootPart = Character:WaitForChild("HumanoidRootPart")

-- Configuration Variables (will be controlled by UI)
local aimbotEnabled = false
local espEnabled = true
local showName = true
local showHealth = true
local showDistance = false
local AIM_SPEED = 0.1
local FOV = 90

-- Aimbot Functions
local function getEnemies()
    local enemies = {}
    local localPlayerTeam = LocalPlayer.Team

    for _, player in ipairs(Players:GetPlayers()) do
        if player ~= LocalPlayer and player.Character and player.Character:FindFirstChild("Humanoid") and player.Character.Humanoid.Health > 0 then
            if localPlayerTeam and player.Team and player.Team ~= localPlayerTeam then
                table.insert(enemies, player)
            elseif not localPlayerTeam and not player.Team then
                table.insert(enemies, player)
            end
        end
    end
    return enemies
end

local function findTarget()
    local closestTarget = nil
    local minDistance = math.huge
    local localPlayerPosition = RootPart.Position

    for _, enemyPlayer in ipairs(getEnemies()) do
        local enemyCharacter = enemyPlayer.Character
        local enemyRootPart = enemyCharacter:FindFirstChild("HumanoidRootPart")

        if enemyRootPart then
            local distance = (localPlayerPosition - enemyRootPart.Position).Magnitude
            local direction = (enemyRootPart.Position - localPlayerPosition).Unit
            local lookVector = RootPart.CFrame.LookVector

            local angle = math.deg(math.acos(lookVector:Dot(direction)))

            if angle < FOV / 2 and distance < minDistance then
                minDistance = distance
                closestTarget = enemyRootPart
            end
        end
    end
    return closestTarget
end

-- ESP Functions
local espLabels = {}

local function createESPLabel(player)
    local label = Instance.new("TextLabel")
    label.Name = player.Name .. "ESPLabel"
    label.Size = UDim2.new(0, 150, 0, 20)
    label.BackgroundTransparency = 1
    label.TextScaled = true
    label.TextColor3 = Color3.new(1, 1, 1)
    label.TextStrokeTransparency = 0
    label.Font = Enum.Font.SourceSansBold
    label.ZIndex = 10
    label.Parent = espScreenGui -- Parent to ScreenGui directly for overlay
    return label
end

local function updateESP()
    local localPlayerTeam = LocalPlayer.Team

    for _, player in ipairs(Players:GetPlayers()) do
        if player ~= LocalPlayer and player.Character and player.Character:FindFirstChild("Humanoid") and player.Character.Humanoid.Health > 0 then
            local isEnemy = false
            if localPlayerTeam and player.Team and player.Team ~= localPlayerTeam then
                isEnemy = true
            elseif not localPlayerTeam and not player.Team then
                isEnemy = true
            end

            if isEnemy then
                local character = player.Character
                local humanoid = character:FindFirstChild("Humanoid")
                local rootPart = character:FindFirstChild("HumanoidRootPart")

                if humanoid and rootPart then
                    local screenPosition, onScreen = workspace.CurrentCamera:WorldToScreenPoint(rootPart.Position)

                    local label = espLabels[player.UserId]
                    if not label then
                        label = createESPLabel(player)
                        espLabels[player.UserId] = label
                    end

                    if onScreen and espEnabled then
                        label.Position = UDim2.new(0, screenPosition.X - label.AbsoluteSize.X / 2, 0, screenPosition.Y - label.AbsoluteSize.Y / 2)
                        local text = ""
                        if showName then text = text .. player.Name end
                        if showHealth then text = text .. string.format(" (%.0f HP)", humanoid.Health) end
                        if showDistance then text = text .. string.format(" [%.0f studs]", (RootPart.Position - rootPart.Position).Magnitude) end
                        label.Text = text
                        label.Visible = true
                    else
                        label.Visible = false
                    end
                end
            else
                if espLabels[player.UserId] then
                    espLabels[player.UserId]:Destroy()
                    espLabels[player.UserId] = nil
                end
            end
        else
            if espLabels[player.UserId] then
                espLabels[player.UserId]:Destroy()
                espLabels[player.UserId] = nil
            end
        end
    end
end

-- Main RenderStepped loop for Aimbot and ESP
RunService.RenderStepped:Connect(function()
    if Humanoid.Health <= 0 then return end

    if aimbotEnabled then
        local target = findTarget()
        if target then
            local targetPosition = target.Position
            local currentCFrame = RootPart.CFrame
            local lookAt = CFrame.new(currentCFrame.Position, targetPosition)
            RootPart.CFrame = currentCFrame:Lerp(lookAt, AIM_SPEED)
        end
    end

    if espEnabled then
        updateESP()
    else
        for _, label in pairs(espLabels) do
            label.Visible = false
        end
    end
end)

print("ESP and Aimbot core logic integrated.")

-- UI Interaction Logic (to be added in next steps)

-- Toggle ESP Button Click
espToggleButton.MouseButton1Click:Connect(function()
    espEnabled = not espEnabled
    espToggleButton.Text = "Toggle ESP: " .. (espEnabled and "ON" or "OFF")
    espScreenGui.Enabled = espEnabled -- Control visibility of ESP labels
end)

-- Toggle Aimbot Button Click
aimbotToggleButton.MouseButton1Click:Connect(function()
    aimbotEnabled = not aimbotEnabled
    aimbotToggleButton.Text = "Toggle Aimbot: " .. (aimbotEnabled and "ON" or "OFF")
end)

-- ESP Options (Name, Health, Distance)
local nameToggle = Instance.new("TextButton")
nameToggle.Name = "NameToggle"
nameToggle.Size = UDim2.new(0.9, 0, 0, 25)
nameToggle.Position = UDim2.new(0.05, 0, 0, 5)
nameToggle.BackgroundColor3 = Color3.fromRGB(70, 70, 70)
nameToggle.TextColor3 = Color3.fromRGB(255, 255, 255)
nameToggle.Text = "Show Name: ON"
nameToggle.Font = Enum.Font.SourceSans
nameToggle.TextScaled = true
nameToggle.Parent = espOptionsFrame

nameToggle.MouseButton1Click:Connect(function()
    showName = not showName
    nameToggle.Text = "Show Name: " .. (showName and "ON" or "OFF")
end)

local healthToggle = Instance.new("TextButton")
healthToggle.Name = "HealthToggle"
healthToggle.Size = UDim2.new(0.9, 0, 0, 25)
healthToggle.Position = UDim2.new(0.05, 0, 0, 35)
healthToggle.BackgroundColor3 = Color3.fromRGB(70, 70, 70)
healthToggle.TextColor3 = Color3.fromRGB(255, 255, 255)
healthToggle.Text = "Show Health: ON"
healthToggle.Font = Enum.Font.SourceSans
healthToggle.TextScaled = true
healthToggle.Parent = espOptionsFrame

healthToggle.MouseButton1Click:Connect(function()
    showHealth = not showHealth
    healthToggle.Text = "Show Health: " .. (showHealth and "ON" or "OFF")
end)

local distanceToggle = Instance.new("TextButton")
distanceToggle.Name = "DistanceToggle"
distanceToggle.Size = UDim2.new(0.9, 0, 0, 25)
distanceToggle.Position = UDim2.new(0.05, 0, 0, 65)
distanceToggle.BackgroundColor3 = Color3.fromRGB(70, 70, 70)
distanceToggle.TextColor3 = Color3.fromRGB(255, 255, 255)
distanceToggle.Text = "Show Distance: OFF"
distanceToggle.Font = Enum.Font.SourceSans
distanceToggle.TextScaled = true
distanceToggle.Parent = espOptionsFrame

distanceToggle.MouseButton1Click:Connect(function()
    showDistance = not showDistance
    distanceToggle.Text = "Show Distance: " .. (showDistance and "ON" or "OFF")
end)

-- Aimbot Options (FOV, Aim Speed)
local fovLabel = Instance.new("TextLabel")
fovLabel.Name = "FOVLabel"
fovLabel.Size = UDim2.new(0.4, 0, 0, 25)
fovLabel.Position = UDim2.new(0.05, 0, 0, 5)
fovLabel.BackgroundTransparency = 1
fovLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
fovLabel.Text = "FOV: " .. FOV
fovLabel.Font = Enum.Font.SourceSans
fovLabel.TextScaled = true
fovLabel.TextXAlignment = Enum.TextXAlignment.Left
fovLabel.Parent = aimbotOptionsFrame

local fovTextBox = Instance.new("TextBox")
fovTextBox.Name = "FOVTextBox"
fovTextBox.Size = UDim2.new(0.4, 0, 0, 25)
fovTextBox.Position = UDim2.new(0.55, 0, 0, 5)
fovTextBox.BackgroundColor3 = Color3.fromRGB(70, 70, 70)
fovTextBox.TextColor3 = Color3.fromRGB(255, 255, 255)
fovTextBox.Text = tostring(FOV)
fovTextBox.Font = Enum.Font.SourceSans
fovTextBox.TextScaled = true
fovTextBox.Parent = aimbotOptionsFrame

fovTextBox.FocusLost:Connect(function(enterPressed)
    local newFOV = tonumber(fovTextBox.Text)
    if newFOV and newFOV > 0 and newFOV <= 360 then
        FOV = newFOV
        fovLabel.Text = "FOV: " .. FOV
    else
        fovTextBox.Text = tostring(FOV) -- Revert if invalid
    end
end)

local aimSpeedLabel = Instance.new("TextLabel")
aimSpeedLabel.Name = "AimSpeedLabel"
aimSpeedLabel.Size = UDim2.new(0.4, 0, 0, 25)
aimSpeedLabel.Position = UDim2.new(0.05, 0, 0, 35)
aimSpeedLabel.BackgroundTransparency = 1
aimSpeedLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
aimSpeedLabel.Text = "Aim Speed: " .. AIM_SPEED
aimSpeedLabel.Font = Enum.Font.SourceSans
aimSpeedLabel.TextScaled = true
aimSpeedLabel.TextXAlignment = Enum.TextXAlignment.Left
aimSpeedLabel.Parent = aimbotOptionsFrame

local aimSpeedTextBox = Instance.new("TextBox")
aimSpeedTextBox.Name = "AimSpeedTextBox"
aimSpeedTextBox.Size = UDim2.new(0.4, 0, 0, 25)
aimSpeedTextBox.Position = UDim2.new(0.55, 0, 0, 35)
aimSpeedTextBox.BackgroundColor3 = Color3.fromRGB(70, 70, 70)
aimSpeedTextBox.TextColor3 = Color3.fromRGB(255, 255, 255)
aimSpeedTextBox.Text = tostring(AIM_SPEED)
aimSpeedTextBox.Font = Enum.Font.SourceSans
aimSpeedTextBox.TextScaled = true
aimSpeedTextBox.Parent = aimbotOptionsFrame

aimSpeedTextBox.FocusLost:Connect(function(enterPressed)
    local newAimSpeed = tonumber(aimSpeedTextBox.Text)
    if newAimSpeed and newAimSpeed >= 0 and newAimSpeed <= 1 then
        AIM_SPEED = newAimSpeed
        aimSpeedLabel.Text = "Aim Speed: " .. AIM_SPEED
    else
        aimSpeedTextBox.Text = tostring(AIM_SPEED) -- Revert if invalid
    end
end)

print("UI interaction logic added.")



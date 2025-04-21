--[[
Eggs Farm V1.0
Author: scriptmausrb
Created: 2025-04-21
Updated by: nootmaus
Last update: 2025-04-21 19:21:55 UTC

Description: Auto farm script for collecting coins in game
]]

local Players = game:GetService("Players")
local TweenService = game:GetService("TweenService")
local UserInputService = game:GetService("UserInputService")

local player = Players.LocalPlayer
local playerGui = player:WaitForChild("PlayerGui")

-- Table to store coins that have already been collected (debounce)
local coinDebounce = {}

----------------------------------------------------------------
-- Notification Function
local function showNotification(message)
    local notif = Instance.new("TextLabel")
    notif.Size = UDim2.new(0, 300, 0, 50)
    notif.Position = UDim2.new(0.5, -150, 0, 50)
    notif.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
    notif.BackgroundTransparency = 0.2
    notif.BorderSizePixel = 0
    notif.Text = message
    notif.Font = Enum.Font.GothamBold
    notif.TextSize = 20
    notif.TextColor3 = Color3.new(1,1,1)
    notif.Parent = playerGui

    local uicorner = Instance.new("UICorner", notif)
    uicorner.CornerRadius = UDim.new(0, 10)

    local tween = TweenService:Create(notif, TweenInfo.new(2, Enum.EasingStyle.Linear, Enum.EasingDirection.Out), {BackgroundTransparency = 1, TextTransparency = 1})
    tween:Play()
    tween.Completed:Connect(function()
        notif:Destroy()
    end)
end

----------------------------------------------------------------
-- Create GUI
local screenGui = Instance.new("ScreenGui")
screenGui.Name = "Eggs Farm V1.0"
screenGui.ResetOnSpawn = false
screenGui.Parent = playerGui

local mainFrame = Instance.new("Frame")
mainFrame.Name = "MainFrame"
mainFrame.Size = UDim2.new(0, 200, 0, 120)
mainFrame.Position = UDim2.new(0.5, -100, 0.5, -60)
mainFrame.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
mainFrame.BackgroundTransparency = 0.1
mainFrame.BorderSizePixel = 0
mainFrame.Parent = screenGui
mainFrame.Active = true

local frameCorner = Instance.new("UICorner", mainFrame)
frameCorner.CornerRadius = UDim.new(0, 8)

-- Title
local titleLabel = Instance.new("TextLabel")
titleLabel.Name = "TitleLabel"
titleLabel.Size = UDim2.new(1, -20, 0, 25)
titleLabel.Position = UDim2.new(0, 10, 0, 5)
titleLabel.BackgroundTransparency = 1
titleLabel.Text = "Eggs farm V1.0"
titleLabel.Font = Enum.Font.GothamBold
titleLabel.TextSize = 14
titleLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
titleLabel.TextXAlignment = Enum.TextXAlignment.Left
titleLabel.Parent = mainFrame

-- Author
local authorLabel = Instance.new("TextLabel")
authorLabel.Name = "AuthorLabel"
authorLabel.Size = UDim2.new(1, -20, 0, 15)
authorLabel.Position = UDim2.new(0, 10, 0, 25)
authorLabel.BackgroundTransparency = 1
authorLabel.Text = "by: scriptmausrb"
authorLabel.Font = Enum.Font.GothamSemibold
authorLabel.TextSize = 12
authorLabel.TextColor3 = Color3.fromRGB(200, 200, 200)
authorLabel.TextXAlignment = Enum.TextXAlignment.Left
authorLabel.Parent = mainFrame

-- Buttons Container
local buttonContainer = Instance.new("Frame")
buttonContainer.Name = "ButtonContainer"
buttonContainer.Size = UDim2.new(1, -20, 0, 70)
buttonContainer.Position = UDim2.new(0, 10, 0, 45)
buttonContainer.BackgroundTransparency = 1
buttonContainer.Parent = mainFrame

local listLayout = Instance.new("UIListLayout", buttonContainer)
listLayout.Padding = UDim.new(0, 5)

-- Function to create buttons
local function createButton(text)
    local btn = Instance.new("TextButton")
    btn.Size = UDim2.new(1, 0, 0, 30)
    btn.BackgroundColor3 = Color3.fromRGB(45, 45, 45)
    btn.Text = text
    btn.Font = Enum.Font.GothamSemibold
    btn.TextSize = 14
    btn.TextColor3 = Color3.fromRGB(255, 255, 255)
    btn.BorderSizePixel = 0

    local btnCorner = Instance.new("UICorner", btn)
    btnCorner.CornerRadius = UDim.new(0, 6)

    return btn
end

-- Create main buttons
local autoFarmBtn = createButton("Auto Farm: OFF")
autoFarmBtn.Parent = buttonContainer

local antiAfkBtn = createButton("Anti-AFK: OFF")
antiAfkBtn.Parent = buttonContainer

-- Close button
local closeButton = Instance.new("TextButton")
closeButton.Name = "CloseButton"
closeButton.Size = UDim2.new(0, 20, 0, 20)
closeButton.Position = UDim2.new(1, -25, 0, 5)
closeButton.BackgroundTransparency = 1
closeButton.Text = "×"
closeButton.Font = Enum.Font.GothamBold
closeButton.TextSize = 20
closeButton.TextColor3 = Color3.fromRGB(200, 200, 200)
closeButton.Parent = mainFrame

----------------------------------------------------------------
-- Auto Farm Code
local mapsList = {
    "Factory", "BioLab", "House2", "Hospital3", "Workplace",
    "MilBase", "Bank2", "Hotel2", "Mansion2", "Office3",
    "PoliceStation", "ResearchFacility", "Hotel", "VampireCastle"
}

-- Функция проверки лобби
local function isInLobby()
    local lobbyMap = workspace:FindFirstChild("Lobby")
    if lobbyMap and player.Character then
        local humanoidRootPart = player.Character:FindFirstChild("HumanoidRootPart")
        if humanoidRootPart then
            for _, part in pairs(lobbyMap:GetDescendants()) do
                if part:IsA("BasePart") and 
                   (humanoidRootPart.Position - part.Position).Magnitude < 100 then
                    return true
                end
            end
        end
    end
    return false
end

-- Функция очистки дебаунса
local function clearCoinDebounce()
    table.clear(coinDebounce)
end

local function getAllCoins()
    local allCoins = {}
    for _, mapName in ipairs(mapsList) do
        local mapFolder = workspace:FindFirstChild(mapName)
        if mapFolder then
            local coinContainer = mapFolder:FindFirstChild("CoinContainer")
            if coinContainer then
                for _, coin in ipairs(coinContainer:GetChildren()) do
                    if coin:FindFirstChild("CoinVisual") and not coinDebounce[coin.CoinVisual] then
                        table.insert(allCoins, coin.CoinVisual)
                    end
                end
            end
        end
    end
    return allCoins
end

local function getClosestCoin(humanoidRootPart)
    local coins = getAllCoins()
    local closestCoin, closestDistance = nil, math.huge
    for _, coin in ipairs(coins) do
        if coin and coin.Parent then
            local distance = (humanoidRootPart.Position - coin.Position).Magnitude
            if distance < closestDistance then
                closestDistance = distance
                closestCoin = coin
            end
        end
    end
    return closestCoin
end

-- Новая функция для проверки раунда
local function checkRound()
    clearCoinDebounce()
    wait(1)
    return true
end

local function tweenToCoin(humanoidRootPart, coin)
    if not (humanoidRootPart and coin and coin.Parent) then return end

    local targetPos = Vector3.new(coin.Position.X, humanoidRootPart.Position.Y, coin.Position.Z)
    local distance = (humanoidRootPart.Position - targetPos).Magnitude
    local walkSpeed = 18 -- Увеличенная скорость
    local tweenTime = distance / (walkSpeed * 1.2)

    local success, _ = pcall(function()
        local tweenInfo = TweenInfo.new(tweenTime, Enum.EasingStyle.Linear, Enum.EasingDirection.InOut)
        local tween = TweenService:Create(humanoidRootPart, tweenInfo, {CFrame = CFrame.new(targetPos)})
        tween:Play()
        tween.Completed:Wait()

        if coin and coin:FindFirstChild("TouchInterest") then
            firetouchinterest(humanoidRootPart, coin, 0)
            wait(0.1)
            firetouchinterest(humanoidRootPart, coin, 1)
        end
    end)

    if not success then
        wait(0.1)
    end
end

-- Глобальные переменные для фарма
local autoFarmEnabled = false
local farmThread = nil
local characterConnection = nil

-- Функция фарма для персонажа
local function farmOnCharacter(character)
    if not character then return end
    
    local humanoidRootPart = character:WaitForChild("HumanoidRootPart")
    local humanoid = character:WaitForChild("Humanoid")
    
    humanoid.Died:Connect(function()
        clearCoinDebounce()
        wait(3)
        checkRound()
    end)
    
    while autoFarmEnabled do
        -- Проверяем, находится ли игрок в лобби
        if isInLobby() then
            wait(1)
            continue
        end
        
        if not character:IsDescendantOf(game.Workspace) then
            wait(1)
            continue
        end
        
        if humanoid.Health <= 0 then
            wait(3)
            checkRound()
            continue
        end

        local coin = getClosestCoin(humanoidRootPart)
        if coin then
            pcall(function()
                tweenToCoin(humanoidRootPart, coin)
                coinDebounce[coin] = true
            end)
            wait(0.1)
        else
            checkRound()
            wait(0.05)
        end
    end
end

-- Основная функция автофарма
local function startAutoFarm()
    if characterConnection then
        characterConnection:Disconnect()
    end
    
    local function onCharacterAdded(newCharacter)
        if farmThread then
            coroutine.close(farmThread)
        end
        
        farmThread = coroutine.create(function()
            farmOnCharacter(newCharacter)
        end)
        coroutine.resume(farmThread)
    end
    
    characterConnection = player.CharacterAdded:Connect(onCharacterAdded)
    
    if player.Character then
        onCharacterAdded(player.Character)
    end
end

----------------------------------------------------------------
-- Anti-AFK Code
local antiAfkEnabled = false
local VirtualUser = game:GetService("VirtualUser")
local antiAfkConnection

local function enableAntiAfk()
    antiAfkConnection = player.Idled:Connect(function()
        VirtualUser:CaptureController()
        VirtualUser:ClickButton2(Vector2.new(0, 0))
    end)
end

local function disableAntiAfk()
    if antiAfkConnection then
        antiAfkConnection:Disconnect()
        antiAfkConnection = nil
    end
end

----------------------------------------------------------------
-- Button Connections
autoFarmBtn.MouseButton1Click:Connect(function()
    autoFarmEnabled = not autoFarmEnabled
    if autoFarmEnabled then
        autoFarmBtn.Text = "Auto Farm: ON"
        autoFarmBtn.BackgroundColor3 = Color3.fromRGB(60, 60, 60)
        startAutoFarm()
        showNotification("Auto Farm включен")
    else
        autoFarmBtn.Text = "Auto Farm: OFF"
        autoFarmBtn.BackgroundColor3 = Color3.fromRGB(45, 45, 45)
        
        if farmThread then
            coroutine.close(farmThread)
            farmThread = nil
        end
        
        if characterConnection then
            characterConnection:Disconnect()
            characterConnection = nil
        end
        
        showNotification("Auto Farm выключен")
    end
end)

antiAfkBtn.MouseButton1Click:Connect(function()
    antiAfkEnabled = not antiAfkEnabled
    if antiAfkEnabled then
        antiAfkBtn.Text = "Anti-AFK: ON"
        antiAfkBtn.BackgroundColor3 = Color3.fromRGB(60, 60, 60)
        enableAntiAfk()
        showNotification("Anti-AFK включен")
    else
        antiAfkBtn.Text = "Anti-AFK: OFF"
        antiAfkBtn.BackgroundColor3 = Color3.fromRGB(45, 45, 45)
        disableAntiAfk()
        showNotification("Anti-AFK выключен")
    end
end)

closeButton.MouseButton1Click:Connect(function()
    screenGui:Destroy()
end)

----------------------------------------------------------------
-- Drag Functionality
local dragging, dragInput, dragStart, startPos

local function update(input)
    local delta = input.Position - dragStart
    mainFrame.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X,
                                   startPos.Y.Scale, startPos.Y.Offset + delta.Y)
end

mainFrame.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        dragging = true
        dragStart = input.Position
        startPos = mainFrame.Position
        
        input.Changed:Connect(function()
            if input.UserInputState == Enum.UserInputState.End then
                dragging = false
            end
        end)
    end
end)

mainFrame.InputChanged:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseMovement then
        dragInput = input
    end
end)

UserInputService.InputChanged:Connect(function(input)
    if input == dragInput and dragging then
        update(input)
    end
end)

print("Eggs Farm V1.0 loaded successfully!")

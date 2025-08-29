-- Roblox 控制面板完整脚本（含玩家区域传送功能）
-- 支持玩家 Henrietta (knsidhqk) 的传送、速度修改和玩家传送功能
if not game:IsLoaded() then
    game.Loaded:Wait()
end

-- 等待玩家加载完成
local Players = game:GetService("Players")
local player = Players.LocalPlayer
while not player do
    wait(0.1)
    player = Players.LocalPlayer
end

-- 确保CoreGui已加载
local CoreGui = game:GetService("CoreGui")
local UserInputService = game:GetService("UserInputService")
local RunService = game:GetService("RunService")

-- 存储传送状态和循环
local teleportLoops = {
    Meat_1 = false, Meat_2 = false, Meat_3 = false, Meat_4 = false,
    Meat_5 = false, Meat_6 = false, Meat_7 = false,
    Egg = false,
    BigEgg = false
}

-- 存储原始速度值
local originalWalkSpeed = 16
local originalJumpPower = 50

-- 存储玩家列表
local playerList = {}
local playerButtons = {}

-- 创建主界面 - 调整位置向下
local function createControlPanel()
    -- 先检查是否已存在同名的ScreenGui，避免重复创建
    if CoreGui:FindFirstChild("EnhancedControlPanel") then
        CoreGui:FindFirstChild("EnhancedControlPanel"):Destroy()
    end
    
    local screenGui = Instance.new("ScreenGui")
    screenGui.Name = "EnhancedControlPanel"
    screenGui.Parent = CoreGui
    screenGui.ResetOnSpawn = false
    screenGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
    
    -- 调整位置向下
    local mainFrame = Instance.new("Frame")
    mainFrame.Size = UDim2.new(0, 400, 0, 600)  -- 增加宽度以容纳新区域
    mainFrame.Position = UDim2.new(0.5, -200, 0.6, -250)
    mainFrame.AnchorPoint = Vector2.new(0.5, 0.5)
    mainFrame.BackgroundColor3 = Color3.fromRGB(40, 40, 45)
    mainFrame.BorderSizePixel = 0
    mainFrame.Parent = screenGui
    
    -- 添加圆角效果
    local uiCorner = Instance.new("UICorner")
    uiCorner.CornerRadius = UDim.new(0, 8)
    uiCorner.Parent = mainFrame
    
    -- 标题栏
    local titleBar = Instance.new("Frame")
    titleBar.Size = UDim2.new(1, 0, 0, 30)
    titleBar.Position = UDim2.new(0, 0, 0, 0)
    titleBar.BackgroundColor3 = Color3.fromRGB(30, 30, 35)
    titleBar.BorderSizePixel = 0
    titleBar.Parent = mainFrame

    local titleCorner = Instance.new("UICorner")
    titleCorner.CornerRadius = UDim.new(0, 8)
    titleCorner.Parent = titleBar
    
    local titleText = Instance.new("TextLabel")
    titleText.Size = UDim2.new(1, -40, 1, 0)
    titleText.Position = UDim2.new(0, 10, 0, 0)
    titleText.BackgroundTransparency = 1
    titleText.TextColor3 = Color3.fromRGB(255, 255, 255)
    titleText.Text = "Henrietta 控制面板"
    titleText.Font = Enum.Font.GothamBold
    titleText.TextSize = 14
    titleText.TextXAlignment = Enum.TextXAlignment.Left
    titleText.Parent = titleBar
    
    local closeButton = Instance.new("TextButton")
    closeButton.Size = UDim2.new(0, 30, 0, 30)
    closeButton.Position = UDim2.new(1, -30, 0, 0)
    closeButton.BackgroundColor3 = Color3.fromRGB(200, 50, 50)
    closeButton.BorderSizePixel = 0
    closeButton.TextColor3 = Color3.fromRGB(255, 255, 255)
    closeButton.Text = "X"
    closeButton.Font = Enum.Font.GothamBold
    closeButton.TextSize = 14
    closeButton.Parent = titleBar
    
    local closeButtonCorner = Instance.new("UICorner")
    closeButtonCorner.CornerRadius = UDim.new(0, 6)
    closeButtonCorner.Parent = closeButton
    
    -- 内容区域
    local scrollFrame = Instance.new("ScrollingFrame")
    scrollFrame.Size = UDim2.new(1, -10, 1, -40)
    scrollFrame.Position = UDim2.new(0, 5, 0, 35)
    scrollFrame.BackgroundTransparency = 1
    scrollFrame.ScrollBarThickness = 6
    scrollFrame.CanvasSize = UDim2.new(0, 0, 0, 0)
    scrollFrame.Parent = mainFrame
    
    local layout = Instance.new("UIListLayout")
    layout.Padding = UDim.new(0, 10)
    layout.Parent = scrollFrame
    
    -- 玩家区域 - 新增
    local playersSection = Instance.new("Frame")
    playersSection.Size = UDim2.new(1, 0, 0, 120)
    playersSection.BackgroundColor3 = Color3.fromRGB(45, 45, 65)
    playersSection.BorderSizePixel = 0
    playersSection.Parent = scrollFrame
    playersSection.Name = "PlayersSection"
    
    local playersSectionCorner = Instance.new("UICorner")
    playersSectionCorner.CornerRadius = UDim.new(0, 6)
    playersSectionCorner.Parent = playersSection
    
    local playersTitle = Instance.new("TextLabel")
    playersTitle.Size = UDim2.new(1, 0, 0, 25)
    playersTitle.Position = UDim2.new(0, 0, 0, 0)
    playersTitle.BackgroundColor3 = Color3.fromRGB(55, 55, 80)
    playersTitle.BorderSizePixel = 0
    playersTitle.TextColor3 = Color3.fromRGB(200, 200, 255)
    playersTitle.Text = "玩家传送 (点击名字传送)"
    playersTitle.Font = Enum.Font.GothamBold
    playersTitle.TextSize = 14
    playersTitle.Parent = playersSection
    
    local playersScrollFrame = Instance.new("ScrollingFrame")
    playersScrollFrame.Size = UDim2.new(1, -10, 0, 85)
    playersScrollFrame.Position = UDim2.new(0, 5, 0, 30)
    playersScrollFrame.BackgroundTransparency = 1
    playersScrollFrame.ScrollBarThickness = 4
    playersScrollFrame.CanvasSize = UDim2.new(0, 0, 0, 0)
    playersScrollFrame.Parent = playersSection
    
    local playersLayout = Instance.new("UIListLayout")
    playersLayout.Padding = UDim.new(0, 5)
    playersLayout.Parent = playersScrollFrame
    
    -- 速度修改区域
    local speedSection = Instance.new("Frame")
    speedSection.Size = UDim2.new(1, 0, 0, 100)
    speedSection.BackgroundColor3 = Color3.fromRGB(45, 45, 60)
    speedSection.BorderSizePixel = 0
    speedSection.Parent = scrollFrame
    
    local speedSectionCorner = Instance.new("UICorner")
    speedSectionCorner.CornerRadius = UDim.new(0, 6)
    speedSectionCorner.Parent = speedSection
    
    local speedTitle = Instance.new("TextLabel")
    speedTitle.Size = UDim2.new(1, 0, 0, 25)
    speedTitle.Position = UDim2.new(0, 0, 0, 0)
    speedTitle.BackgroundColor3 = Color3.fromRGB(55, 55, 75)
    speedTitle.BorderSizePixel = 0
    speedTitle.TextColor3 = Color3.fromRGB(200, 200, 255)
    speedTitle.Text = "速度修改"
    speedTitle.Font = Enum.Font.GothamBold
    speedTitle.TextSize = 14
    speedTitle.Parent = speedSection
    
    local speedInput = Instance.new("TextBox")
    speedInput.Size = UDim2.new(0, 120, 0, 30)
    speedInput.Position = UDim2.new(0, 10, 0, 30)
    speedInput.BackgroundColor3 = Color3.fromRGB(60, 60, 80)
    speedInput.BorderSizePixel = 0
    speedInput.TextColor3 = Color3.fromRGB(255, 255, 255)
    speedInput.Text = "50"
    speedInput.PlaceholderText = "输入速度值"
    speedInput.Font = Enum.Font.Gotham
    speedInput.TextSize = 14
    speedInput.Parent = speedSection
    
    local inputCorner = Instance.new("UICorner")
    inputCorner.CornerRadius = UDim.new(0, 4)
    inputCorner.Parent = speedInput
    
    local applySpeedButton = Instance.new("TextButton")
    applySpeedButton.Size = UDim2.new(0, 80, 0, 30)
    applySpeedButton.Position = UDim2.new(0, 140, 0, 30)
    applySpeedButton.BackgroundColor3 = Color3.fromRGB(60, 120, 60)
    applySpeedButton.BorderSizePixel = 0
    applySpeedButton.TextColor3 = Color3.fromRGB(255, 255, 255)
    applySpeedButton.Text = "应用速度"
    applySpeedButton.Font = Enum.Font.Gotham
    applySpeedButton.TextSize = 12
    applySpeedButton.Parent = speedSection
    
    local applyButtonCorner = Instance.new("UICorner")
    applyButtonCorner.CornerRadius = UDim.new(0, 4)
    applyButtonCorner.Parent = applySpeedButton
    
    local resetSpeedButton = Instance.new("TextButton")
    resetSpeedButton.Size = UDim2.new(0, 80, 0, 30)
    resetSpeedButton.Position = UDim2.new(0, 230, 0, 30)
    resetSpeedButton.BackgroundColor3 = Color3.fromRGB(120, 60, 60)
    resetSpeedButton.BorderSizePixel = 0
    resetSpeedButton.TextColor3 = Color3.fromRGB(255, 255, 255)
    resetSpeedButton.Text = "重置速度"
    resetSpeedButton.Font = Enum.Font.Gotham
    resetSpeedButton.TextSize = 12
    resetSpeedButton.Parent = speedSection
    
    local resetButtonCorner = Instance.new("UICorner")
    resetButtonCorner.CornerRadius = UDim.new(0, 4)
    resetButtonCorner.Parent = resetSpeedButton
    
    local speedStatus = Instance.new("TextLabel")
    speedStatus.Size = UDim2.new(1, -20, 0, 20)
    speedStatus.Position = UDim2.new(0, 10, 0, 65)
    speedStatus.BackgroundTransparency = 1
    speedStatus.TextColor3 = Color3.fromRGB(200, 200, 200)
    speedStatus.Text = "当前速度: 16"
    speedStatus.Font = Enum.Font.Gotham
    speedStatus.TextSize = 12
    speedStatus.TextXAlignment = Enum.TextXAlignment.Left
    speedStatus.Parent = speedSection
    
    -- 肉类区域
    local meatSection = Instance.new("Frame")
    meatSection.Size = UDim2.new(1, 0, 0, 150)
    meatSection.BackgroundColor3 = Color3.fromRGB(60, 40, 40)
    meatSection.BorderSizePixel = 0
    meatSection.Parent = scrollFrame
    
    local meatSectionCorner = Instance.new("UICorner")
    meatSectionCorner.CornerRadius = UDim.new(0, 6)
    meatSectionCorner.Parent = meatSection
    
    local meatTitle = Instance.new("TextLabel")
    meatTitle.Size = UDim2.new(1, 0, 0, 25)
    meatTitle.Position = UDim2.new(0, 0, 0, 0)
    meatTitle.BackgroundColor3 = Color3.fromRGB(80, 50, 50)
    meatTitle.BorderSizePixel = 0
    meatTitle.TextColor3 = Color3.fromRGB(255, 200, 200)
    meatTitle.Text = "肉类传送"
    meatTitle.Font = Enum.Font.GothamBold
    meatTitle.TextSize = 14
    meatTitle.Parent = meatSection
    
    local meatGrid = Instance.new("UIGridLayout")
    meatGrid.CellSize = UDim2.new(0, 70, 0, 30)
    meatGrid.CellPadding = UDim2.new(0, 5, 0, 5)
    meatGrid.StartCorner = Enum.StartCorner.TopLeft
    meatGrid.HorizontalAlignment = Enum.HorizontalAlignment.Center
    meatGrid.VerticalAlignment = Enum.VerticalAlignment.Center
    meatGrid.SortOrder = Enum.SortOrder.LayoutOrder
    meatGrid.Parent = meatSection
    
    -- 创建肉类按钮
    for i = 1, 7 do
        local buttonName = "Meat_" .. i
        local button = Instance.new("TextButton")
        button.Size = UDim2.new(0, 70, 0, 30)
        button.BackgroundColor3 = Color3.fromRGB(100, 60, 60)
        button.BorderSizePixel = 0
        button.TextColor3 = Color3.fromRGB(255, 255, 255)
        button.Text = buttonName
        button.Font = Enum.Font.Gotham
        button.TextSize = 12
        button.LayoutOrder = i
        button.Parent = meatSection
        
        local buttonCorner = Instance.new("UICorner")
        buttonCorner.CornerRadius = UDim.new(0, 4)
        buttonCorner.Parent = button
        
        button.MouseButton1Click:Connect(function()
            teleportLoops[buttonName] = not teleportLoops[buttonName]
            button.BackgroundColor3 = teleportLoops[buttonName] and 
                Color3.fromRGB(60, 100, 60) or Color3.fromRGB(100, 60, 60)
            print(buttonName .. " 状态: " .. (teleportLoops[buttonName] and "开启" or "关闭"))
        end)
    end
    
    -- 鸡蛋区域
    local eggSection = Instance.new("Frame")
    eggSection.Size = UDim2.new(1, 0, 0, 60)
    eggSection.BackgroundColor3 = Color3.fromRGB(40, 60, 40)
    eggSection.BorderSizePixel = 0
    eggSection.Parent = scrollFrame
    
    local eggSectionCorner = Instance.new("UICorner")
    eggSectionCorner.CornerRadius = UDim.new(0, 6)
    eggSectionCorner.Parent = eggSection
    
    local eggTitle = Instance.new("TextLabel")
    eggTitle.Size = UDim2.new(1, 0, 0, 25)
    eggTitle.Position = UDim2.new(0, 0, 0, 0)
    eggTitle.BackgroundColor3 = Color3.fromRGB(50, 80, 50)
    eggTitle.BorderSizePixel = 0
    eggTitle.TextColor3 = Color3.fromRGB(200, 255, 200)
    eggTitle.Text = "鸡蛋传送"
    eggTitle.Font = Enum.Font.GothamBold
    eggTitle.TextSize = 14
    eggTitle.Parent = eggSection
    
    local eggButton = Instance.new("TextButton")
    eggButton.Size = UDim2.new(0, 100, 0, 30)
    eggButton.Position = UDim2.new(0.5, -50, 0, 30)
    eggButton.BackgroundColor3 = Color3.fromRGB(60, 100, 60)
    eggButton.BorderSizePixel = 0
    eggButton.TextColor3 = Color3.fromRGB(255, 255, 255)
    eggButton.Text = "Egg"
    eggButton.Font = Enum.Font.Gotham
    eggButton.TextSize = 12
    eggButton.Parent = eggSection
    
    local eggButtonCorner = Instance.new("UICorner")
    eggButtonCorner.CornerRadius = UDim.new(0, 4)
    eggButtonCorner.Parent = eggButton
    
    eggButton.MouseButton1Click:Connect(function()
        teleportLoops.Egg = not teleportLoops.Egg
        eggButton.BackgroundColor3 = teleportLoops.Egg and 
            Color3.fromRGB(60, 100, 60) or Color3.fromRGB(100, 60, 60)
        print("Egg 状态: " .. (teleportLoops.Egg and "开启" or "关闭"))
    end)
    
    -- 鸡蛋群区域
    local bigEggSection = Instance.new("Frame")
    bigEggSection.Size = UDim2.new(1, 0, 0, 60)
    bigEggSection.BackgroundColor3 = Color3.fromRGB(40, 40, 60)
    bigEggSection.BorderSizePixel = 0
    bigEggSection.Parent = scrollFrame
    
    local bigEggSectionCorner = Instance.new("UICorner")
    bigEggSectionCorner.CornerRadius = UDim.new(0, 6)
    bigEggSectionCorner.Parent = bigEggSection
    
    local bigEggTitle = Instance.new("TextLabel")
    bigEggTitle.Size = UDim2.new(1, 0, 0, 25)
    bigEggTitle.Position = UDim2.new(0, 0, 0, 0)
    bigEggTitle.BackgroundColor3 = Color3.fromRGB(50, 50, 80)
    bigEggTitle.BorderSizePixel = 0
    bigEggTitle.TextColor3 = Color3.fromRGB(200, 200, 255)
    bigEggTitle.Text = "鸡蛋群传送"
    bigEggTitle.Font = Enum.Font.GothamBold
    bigEggTitle.TextSize = 14
    bigEggTitle.Parent = bigEggSection
    
    local bigEggButton = Instance.new("TextButton")
    bigEggButton.Size = UDim2.new(0, 100, 0, 30)
    bigEggButton.Position = UDim2.new(0.5, -50, 0, 30)
    bigEggButton.BackgroundColor3 = Color3.fromRGB(60, 60, 100)
    bigEggButton.BorderSizePixel = 0
    bigEggButton.TextColor3 = Color3.fromRGB(255, 255, 255)
    bigEggButton.Text = "BigEgg"
    bigEggButton.Font = Enum.Font.Gotham
    bigEggButton.TextSize = 12
    bigEggButton.Parent = bigEggSection
    
    local bigEggButtonCorner = Instance.new("UICorner")
    bigEggButtonCorner.CornerRadius = UDim.new(0, 4)
    bigEggButtonCorner.Parent = bigEggButton
    
    bigEggButton.MouseButton1Click:Connect(function()
        teleportLoops.BigEgg = not teleportLoops.BigEgg
        bigEggButton.BackgroundColor3 = teleportLoops.BigEgg and 
            Color3.fromRGB(60, 100, 60) or Color3.fromRGB(60, 60, 100)
        print("BigEgg 状态: " .. (teleportLoops.BigEgg and "开启" or "关闭"))
    end)
    
    -- 更新滚动区域大小
    layout:GetPropertyChangedSignal("AbsoluteContentSize"):Connect(function()
        scrollFrame.CanvasSize = UDim2.new(0, 0, 0, layout.AbsoluteContentSize.Y)
    end)
    
    playersLayout:GetPropertyChangedSignal("AbsoluteContentSize"):Connect(function()
        playersScrollFrame.CanvasSize = UDim2.new(0, 0, 0, playersLayout.AbsoluteContentSize.Y)
    end)
    
    meatGrid:GetPropertyChangedSignal("AbsoluteContentSize"):Connect(function()
        meatSection.Size = UDim2.new(1, 0, 0, meatGrid.AbsoluteContentSize.Y + 30)
    end)
    
    -- 使窗口可拖动
    local dragging, dragInput, dragStart, startPos
    
    local function update(input)
        local delta = input.Position - dragStart
        mainFrame.Position = UDim2.new(
            startPos.X.Scale, 
            startPos.X.Offset + delta.X, 
            startPos.Y.Scale, 
            startPos.Y.Offset + delta.Y
        )
    end
    
    titleBar.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 or 
           input.UserInputType == Enum.UserInputType.Touch then
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
    
    titleBar.InputChanged:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseMovement or 
           input.UserInputType == Enum.UserInputType.Touch then
            dragInput = input
        end
    end)
    
    UserInputService.InputChanged:Connect(function(input)
        if input == dragInput and dragging then
            update(input)
        end
    end)
    
    -- 关闭按钮功能
    closeButton.MouseButton1Click:Connect(function()
        screenGui:Destroy()
    end)
    
    -- 速度修改功能
    local function modifySpeed(speedValue)
        local player = findPlayer()
        if not player then
            warn("未找到玩家: knsidhqk (Henrietta)")
            return false
        end
        
        local character = getPlayerCharacter(player)
        if not character then
            warn("玩家没有角色或角色尚未加载完成")
            return false
        end
        
        local humanoid = character:FindFirstChildOfClass("Humanoid")
        if not humanoid then
            warn("角色没有Humanoid")
            return false
        end
        
        -- 设置新的速度值
        humanoid.WalkSpeed = speedValue
        speedStatus.Text = "当前速度: " .. tostring(speedValue)
        print("已设置速度为: " .. tostring(speedValue))
        return true
    end
    
    -- 重置速度功能
    local function resetSpeed()
        local player = findPlayer()
        if not player then return false end
        
        local character = getPlayerCharacter(player)
        if not character then return false end
        
        local humanoid = character:FindFirstChildOfClass("Humanoid")
        if not humanoid then return false end
        
        -- 重置为默认速度
        humanoid.WalkSpeed = originalWalkSpeed
        humanoid.JumpPower = originalJumpPower
        speedStatus.Text = "当前速度: " .. tostring(originalWalkSpeed)
        speedInput.Text = tostring(originalWalkSpeed)
        print("已重置速度为默认值: " .. tostring(originalWalkSpeed))
        return true
    end
    
    -- 应用速度按钮功能
    applySpeedButton.MouseButton1Click:Connect(function()
        local speedText = speedInput.Text
        local speedValue = tonumber(speedText)
        
        if speedValue and speedValue >= 0 and speedValue <= 200 then
            modifySpeed(speedValue)
        else
            warn("请输入有效的速度值 (0-200)")
            speedInput.Text = "16"
        end
    end)
    
    -- 重置速度按钮功能
    resetSpeedButton.MouseButton1Click:Connect(function()
        resetSpeed()
    end)
    
    -- 回车键应用速度
    speedInput.FocusLost:Connect(function(enterPressed)
        if enterPressed then
            local speedText = speedInput.Text
            local speedValue = tonumber(speedText)
            
            if speedValue and speedValue >= 0 and speedValue <= 200 then
                modifySpeed(speedValue)
            else
                warn("请输入有效的速度值 (0-200)")
                speedInput.Text = "16"
            end
        end
    end)
    
    print("磁浮窗控制面板已创建!")
    return {
        ScreenGui = screenGui,
        PlayersScrollFrame = playersScrollFrame
    }
end

-- 更新玩家列表函数
local function updatePlayerList()
    local players = Players:GetPlayers()
    local playerListFrame = CoreGui.EnhancedControlPanel.Frame.ScrollingFrame.PlayersSection.PlayersScrollFrame
    
    -- 清空现有玩家列表
    for _, child in ipairs(playerListFrame:GetChildren()) do
        if child:IsA("TextButton") then
            child:Destroy()
        end
    end
    
    -- 添加当前服务器中的所有玩家
    for _, player in ipairs(players) do
        if player.Name ~= "knsidhqk" and player.DisplayName ~= "Henrietta" then
            local playerButton = Instance.new("TextButton")
            playerButton.Size = UDim2.new(1, -10, 0, 25)
            playerButton.BackgroundColor3 = Color3.fromRGB(60, 60, 80)
            playerButton.BorderSizePixel = 0
            playerButton.TextColor3 = Color3.fromRGB(255, 255, 255)
            playerButton.Text = player.Name .. " (" .. player.DisplayName .. ")"
            playerButton.Font = Enum.Font.Gotham
            playerButton.TextSize = 12
            playerButton.TextXAlignment = Enum.TextXAlignment.Left
            playerButton.Parent = playerListFrame
            
            local buttonCorner = Instance.new("UICorner")
            buttonCorner.CornerRadius = UDim.new(0, 4)
            buttonCorner.Parent = playerButton
            
            -- 点击玩家按钮传送到该玩家
            playerButton.MouseButton1Click:Connect(function()
                teleportToPlayer(player)
            end)
            
            -- 存储玩家按钮引用
            playerButtons[player] = playerButton
        end
    end
    
    -- 更新滚动区域大小
    local playerCount = #players - 1  -- 排除自己
    if playerCount > 0 then
        playerListFrame.CanvasSize = UDim2.new(0, 0, 0, playerCount * 30)
    end
end

-- 传送到玩家函数
local function teleportToPlayer(targetPlayer)
    local player = findPlayer()
    if not player then
        warn("未找到当前玩家")
        return false
    end
    
    local character = getPlayerCharacter(player)
    if not character then
        warn("当前玩家没有角色或角色尚未加载完成")
        return false
    end
    
    local targetCharacter = getPlayerCharacter(targetPlayer)
    if not targetCharacter then
        warn("目标玩家没有角色或角色尚未加载完成")
        return false
    end
    
    -- 获取目标玩家位置
    local targetPosition
    if targetCharacter:IsA("Model") and targetCharacter.PrimaryPart then
        targetPosition = targetCharacter.PrimaryPart.Position
    else
        -- 尝试找到目标玩家的HumanoidRootPart或其他BasePart
        local humanoidRootPart = targetCharacter:FindFirstChild("HumanoidRootPart")
        if humanoidRootPart then
            targetPosition = humanoidRootPart.Position
        else
            for _, part in ipairs(targetCharacter:GetDescendants()) do
                if part:IsA("BasePart") then
                    targetPosition = part.Position
                    break
                end
            end
        end
    end
    
    if not targetPosition then
        warn("无法获取目标玩家位置")
        return false
    end
    
    -- 传送当前玩家到目标玩家位置（高度25个单位）
    local success = teleportCharacter(character, targetPosition + Vector3.new(0, 25, 0))
    
    if success then
        print("已传送到玩家: " .. targetPlayer.Name)
    else
        warn("传送到玩家失败")
    end
    
    return success
end

-- 改进的模型查找功能
local function findModel(modelName)
    print("正在查找模型: " .. modelName)
    
    local function searchIn(parent, depth)
        if depth > 10 then return nil end
        
        for _, child in ipairs(parent:GetChildren()) do
            if child.Name == modelName then
                print("找到模型: " .. modelName .. " 在 " .. parent:GetFullName())
                return child
            end
            
            if #child:GetChildren() > 0 then
                local found = searchIn(child, depth + 1)
                if found then return found end
            end
        end
        return nil
    end
    
    local locationsToSearch = {
        workspace,
        game:GetService("ReplicatedStorage"),
        game:GetService("ServerStorage"),
        game:GetService("Lighting")
    }
    
    for _, location in ipairs(locationsToSearch) do
        local foundModel = searchIn(location, 0)
        if foundModel then
            return foundModel
        end
    end
    
    print("警告: 未找到模型 " .. modelName)
    return nil
end

-- 改进的玩家查找功能
local function findPlayer()
    local targetPlayer = Players:FindFirstChild("knsidhqk")
    if targetPlayer then
        return targetPlayer
    end
    
    for _, player in ipairs(Players:GetPlayers()) do
        if player.DisplayName == "Henrietta" then
            return player
        end
    end
    
    if game:GetService("RunService"):IsClient() then
        local localPlayer = Players.LocalPlayer
        if localPlayer then
            return localPlayer
        end
    end
    
    return nil
end

-- 获取玩家角色的函数
local function getPlayerCharacter(player)
    if not player then return nil end
    
    if player.Character then
        return player.Character
    end
    
    local character = player.CharacterAdded:Wait()
    return character
end

-- 传送角色到指定位置的函数
local function teleportCharacter(character, position)
    if not character then return false end
    
    if character:IsA("Model") and character.PrimaryPart then
        character:SetPrimaryPartCFrame(CFrame.new(position))
        return true
    end
    
    local humanoid = character:FindFirstChildOfClass("Humanoid")
    if humanoid then
        humanoid:MoveTo(position)
        return true
    end
    
    for _, part in ipairs(character:GetDescendants()) do
        if part:IsA("BasePart") then
            part.CFrame = CFrame.new(position)
            return true
        end
    end
    
    return false
end

-- 改进的传送功能
local function teleportToModel(modelName)
    local model = findModel(modelName)
    if not model then
        warn("未找到模型: " .. modelName)
        return false
    end
    
    local player = findPlayer()
    if not player then
        warn("未找到玩家: knsidhqk (Henrietta)")
        return false
    end
    
    local character = getPlayerCharacter(player)
    if not character then
        warn("玩家没有角色或角色尚未加载完成")
        return false
    end
    
    local modelPosition
    if model:IsA("Model") and model.PrimaryPart then
        modelPosition = model.PrimaryPart.Position
    elseif model:IsA("BasePart") then
        modelPosition = model.Position
    else
        modelPosition = model:GetPivot().Position
    end
    
    -- 传送高度调整到25个单位
    local targetPosition = modelPosition + Vector3.new(0, 25, 0)
    local success = teleportCharacter(character, targetPosition)
    
    if success then
        print("已传送至: " .. modelName .. "，高度: 25单位")
    else
        warn("传送失败")
    end
    
    return success
end

-- 传送循环
local function startTeleportLoops()
    while true do
        task.wait(0.5)
        
        for i = 1, 7 do
            local buttonName = "Meat_" .. i
            if teleportLoops[buttonName] then
                local success = teleportToModel(buttonName)
                if not success then
                    teleportLoops[buttonName] = false
                    if CoreGui:FindFirstChild("EnhancedControlPanel") then
                        for _, child in ipairs(CoreGui.EnhancedControlPanel.Frame.ScrollingFrame:GetChildren()) do
                            if child.Name == "肉类传送区域" then
                                for _, button in ipairs(child:GetChildren()) do
                                    if button:IsA("TextButton") and button.Text == buttonName then
                                        button.BackgroundColor3 = Color3.fromRGB(100, 60, 60)
                                    end
                                end
                            end
                        end
                    end
                end
            end
        end
        
        if teleportLoops.Egg then
            local success = teleportToModel("DinosaurEgg_Green")
            if not success then
                teleportLoops.Egg = false
                if CoreGui:FindFirstChild("EnhancedControlPanel") then
                    local eggButton = CoreGui.EnhancedControlPanel.Frame.ScrollingFrame:FindFirstChild("EggButton")
                    if eggButton then
                        eggButton.BackgroundColor3 = Color3.fromRGB(100, 60, 60)
                    end
                end
            end
        end
        
        if teleportLoops.BigEgg then
            local success = teleportToModel("Nest_Of_Eggs_Green")
            if not success then
                teleportLoops.BigEgg = false
                if CoreGui:FindFirstChild("EnhancedControlPanel") then
                    local bigEggButton = CoreGui.EnhancedControlPanel.Frame.ScrollingFrame:FindFirstChild("BigEggButton")
                    if bigEggButton then
                        bigEggButton.BackgroundColor3 = Color3.fromRGB(60, 60, 100)
                    end
                end
            end
        end
    end
end

-- 玩家监听循环
local function startPlayerMonitor()
    while true do
        -- 更新玩家列表
        updatePlayerList()
        -- 每5秒更新一次玩家列表
        task.wait(5)
    end
end

-- 创建控制面板
local controlPanel = createControlPanel()

-- 启动传送循环
task.spawn(startTeleportLoops)

-- 启动玩家监控循环
task.spawn(startPlayerMonitor)

print("增强控制面板已加载! 包含传送、速度修改和玩家传送功能。")

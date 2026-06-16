--========================================================================--
--                               tomb.ware                                --
--            Da Lock - High-Performance Mobile Suite Edition             --
--========================================================================--

--// Configuration Layout
local SETTINGS = {
    BoxColor       = Color3.fromRGB(255, 0, 0),    -- Neon Red Accent
    Background     = Color3.fromRGB(10, 10, 10),
    ESPColor       = Color3.fromRGB(0, 255, 255),  -- Cyan Highlights
    
    -- Sound FX
    SparkleSoundID = "rbxassetid://138080514",
    
    -- Core Settings
    MaxTrackRadius = 250,        -- Maximum tracking distance in studs
    
    -- Prediction Physics
    PredictionAmount = 0.135,    -- Look-ahead factor
    UsePartsPrediction = true,

    -- Auto Air Mechanics
    MinVerticalVelocity = 3,     -- Minimum velocity upward to trigger shoot
    FireCooldown        = 0.2    -- Delay between auto-shots
}

--// Services
local Players      = game:GetService("Players")
local TweenService = game:GetService("TweenService")
local Debris       = game:GetService("Debris")
local RunService   = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local LocalPlayer  = Players.LocalPlayer
local Camera       = workspace.CurrentCamera

--// Global States
local isLockActive   = false
local isAutoAirActive = false
local canShoot       = true
local currentTarget  = nil

--========================================================================--
-- 1. STABLE INTRO ENGINE (GUARANTEED FOR EXECUTORS)
--========================================================================--
local function playIntro()
    local PlayerGui = LocalPlayer:WaitForChild("PlayerGui", 10)
    if not PlayerGui then return end
    
    local sg = Instance.new("ScreenGui")
    sg.Name = "tombware_intro"
    sg.ResetOnSpawn = false
    sg.Parent = PlayerGui
    
    local frame = Instance.new("Frame")
    frame.Size = UDim2.new(0, 250, 0, 70)
    frame.Position = UDim2.new(1, 30, 1, -100)
    frame.BackgroundColor3 = SETTINGS.Background
    frame.BorderSizePixel = 0
    frame.Parent = sg
    
    local stroke = Instance.new("UIStroke")
    stroke.Color = SETTINGS.BoxColor
    stroke.Thickness = 2
    stroke.Parent = frame
    
    local label = Instance.new("TextLabel")
    label.Size = UDim2.new(1, 0, 1, 0)
    label.BackgroundTransparency = 1
    label.Text = ""
    label.TextColor3 = Color3.fromRGB(255, 255, 255)
    label.TextSize = 14
    label.Font = Enum.Font.Code
    label.Parent = frame
    
    pcall(function()
        local sound = Instance.new("Sound")
        sound.SoundId = SETTINGS.SparkleSoundID
        sound.Volume = 0.8
        sound.Parent = game:GetService("SoundService")
        sound:Play()
        Debris:AddItem(sound, 3)
    end)
    
    frame:TweenPosition(UDim2.new(1, -280, 1, -100), Enum.EasingDirection.Out, Enum.EasingStyle.Linear, 0.4, true)
    task.wait(0.4)
    
    local text = "script made by tomb.ware | yt: @tomb.s"
    for i = 1, #text do
        label.Text = string.sub(text, 1, i)
        task.wait(0.04)
    end
    
    task.wait(2.5)
    
    frame:TweenPosition(UDim2.new(1, 30, 1, -100), Enum.EasingDirection.In, Enum.EasingStyle.Linear, 0.4, true, function()
        sg:Destroy()
    end)
end

--========================================================================--
-- 2. TARGET TRACER SYSTEM (2D DRAWING ENGINE)
--========================================================================--
local tracerLine = nil
if Drawing then
    tracerLine = Drawing.new("Line")
    tracerLine.Visible = false
    tracerLine.Color = SETTINGS.BoxColor
    tracerLine.Thickness = 1.5
    tracerLine.Transparency = 0.8
end

--========================================================================--
-- 3. CHAMS ESP (VISUAL HIGHLIGHT ENGINE)
--========================================================================--
local espActive = false
local managedHighlights = {}

local function applyESP(player)
    if player == LocalPlayer then return end
    
    local function characterAdded(character)
        if managedHighlights[player] then
            managedHighlights[player]:Destroy()
        end
        
        if espActive then
            local highlight = Instance.new("Highlight")
            highlight.Name = "TombWareHighlight"
            highlight.Adornee = character
            highlight.FillColor = SETTINGS.Background
            highlight.FillTransparency = 0.7
            highlight.OutlineColor = SETTINGS.ESPColor
            highlight.OutlineTransparency = 0.1
            highlight.DepthMode = Enum.HighlightDepthMode.AlwaysOnTop
            highlight.Parent = character
            
            managedHighlights[player] = highlight
        end
    end

    if player.Character then characterAdded(player.Character) end
    player.CharacterAdded:Connect(characterAdded)
end

Players.PlayerAdded:Connect(applyESP)
for _, player in ipairs(Players:GetPlayers()) do applyESP(player) end

local function toggleESP(state)
    espActive = state
    for _, player in ipairs(Players:GetPlayers()) do
        if player.Character then
            local existing = player.Character:FindFirstChild("TombWareHighlight")
            if existing then existing:Destroy() end
            if espActive then applyESP(player) end
        end
    end
end

--========================================================================--
-- 4. COMBAT UTILITY METHODS (AIRBORNE DETECTION & FIRING)
--========================================================================--
local function checkTargetAirborne(target)
    if not target then return false end
    
    local humanoid = target:FindFirstChildOfClass("Humanoid")
    local rootPart = target:FindFirstChild("HumanoidRootPart")
    
    if humanoid and rootPart then
        local state = humanoid:GetState()
        local isAirState = (state == Enum.HumanoidStateType.Jumping or state == Enum.HumanoidStateType.Freefall)
        local isMovingUp = rootPart.AssemblyLinearVelocity.Y > SETTINGS.MinVerticalVelocity
        
        return isAirState or isMovingUp
    end
    return false
end

local function fireEquippedWeapon()
    local character = LocalPlayer.Character
    if not character then return end
    
    local equippedTool = character:FindFirstChildOfClass("Tool")
    if equippedTool then
        equippedTool:Activate()
    end
end

--========================================================================--
-- 5. CAMERA TRACKING MECHANICS & RENDER RUNNER
--========================================================================--
local function getClosestPlayerToCenter()
    local targetCharacter = nil
    local shortestDistance = math.huge
    local screenCenter = Vector2.new(Camera.ViewportSize.X / 2, Camera.ViewportSize.Y / 2)

    for _, player in ipairs(Players:GetPlayers()) do
        if player ~= LocalPlayer and player.Character then
            local root = player.Character:FindFirstChild("HumanoidRootPart")
            local humanoid = player.Character:FindFirstChildOfClass("Humanoid")
            
            if root and humanoid and humanoid.Health > 0 then
                local myRoot = LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart")
                local worldDistance = myRoot and (root.Position - myRoot.Position).Magnitude or math.huge
                
                if worldDistance <= SETTINGS.MaxTrackRadius then
                    local screenPos, onScreen = Camera:WorldToViewportPoint(root.Position)
                    if onScreen then
                        local distanceToCenter = (Vector2.new(screenPos.X, screenPos.Y) - screenCenter).Magnitude
                        if distanceToCenter < shortestDistance then
                            shortestDistance = distanceToCenter
                            targetCharacter = player.Character
                        end
                    end
                end
            end
        end
    end
    return targetCharacter
end

RunService.RenderStepped:Connect(function()
    if isLockActive then
        if not currentTarget or not currentTarget:FindFirstChild("HumanoidRootPart") or (currentTarget:FindFirstChildOfClass("Humanoid") and currentTarget:FindFirstChildOfClass("Humanoid").Health <= 0) then
            currentTarget = getClosestPlayerToCenter()
        end
        
        if currentTarget and currentTarget:FindFirstChild("HumanoidRootPart") then
            local targetPart = currentTarget.HumanoidRootPart
            local targetPosition = targetPart.Position
            
            if SETTINGS.UsePartsPrediction then
                targetPosition = targetPosition + (targetPart.AssemblyLinearVelocity * SETTINGS.PredictionAmount)
            end
            
            -- Absolute Camera Alignment Execution
            Camera.CFrame = CFrame.new(Camera.CFrame.Position, targetPosition)
            
            -- Integrated Auto Air Evaluation Sub-Process
            if isAutoAirActive and canShoot then
                if checkTargetAirborne(currentTarget) then
                    canShoot = false
                    fireEquippedWeapon()
                    task.delay(SETTINGS.FireCooldown, function()
                        canShoot = true
                    end)
                end
            end
            
            -- Render 2D Tracer
            if tracerLine then
                local vector, onScreen = Camera:WorldToViewportPoint(targetPart.Position)
                if onScreen then
                    tracerLine.From = Vector2.new(Camera.ViewportSize.X / 2, Camera.ViewportSize.Y)
                    tracerLine.To = Vector2.new(vector.X, vector.Y)
                    tracerLine.Visible = true
                else
                    tracerLine.Visible = false
                end
            end
        else
            if tracerLine then tracerLine.Visible = false end
        end
    else
        if tracerLine then tracerLine.Visible = false end
    end
end)

--========================================================================--
-- 6. DESKTOP KEYBOARD INPUT (PC OVERRIDES)
--========================================================================--
local airButtonStroke = nil
local airButton = nil

local function toggleAutoAirState()
    isAutoAirActive = not isAutoAirActive
    if airButton and airButtonStroke then
        if isAutoAirActive then
            airButton.Text = "AIR: ON"
            airButton.TextColor3 = Color3.fromRGB(0, 255, 0)
            airButtonStroke.Color = Color3.fromRGB(0, 255, 0)
            airButtonStroke.Thickness = 3
        else
            airButton.Text = "AIR: OFF"
            airButton.TextColor3 = Color3.fromRGB(255, 255, 255)
            airButtonStroke.Color = SETTINGS.BoxColor
            airButtonStroke.Thickness = 2
        end
    end
end

UserInputService.InputBegan:Connect(function(input, gameProcessed)
    if gameProcessed then return end
    if input.KeyCode == Enum.KeyCode.R then
        toggleAutoAirState()
    end
end)

--========================================================================--
-- 7. MOBILE INTERFACE SUITE (WITH MASTER DISAPPEAR SYSTEM)
--========================================================================--
local function setupMobileUI()
    local PlayerGui = LocalPlayer:WaitForChild("PlayerGui")
    
    local gui = Instance.new("ScreenGui", PlayerGui)
    gui.Name = "tombware_mobile_core"
    gui.ResetOnSpawn = false
    
    --// MASTER CONTAINER (Allows grouped visibility toggling)
    local buttonContainer = Instance.new("Frame", gui)
    buttonContainer.Size = UDim2.new(0, 100, 0, 300)
    buttonContainer.Position = UDim2.new(0.78, 0, 0.22, 0)
    buttonContainer.BackgroundTransparency = 1
    buttonContainer.BorderSizePixel = 0
    
    --// DISAPPEAR / MINIMIZE BUTTON (Anchored right above the suite layout)
    local disappearButton = Instance.new("TextButton", gui)
    disappearButton.Size = UDim2.new(0, 70, 0, 35)
    disappearButton.Position = UDim2.new(0.78, 0, 0.14, 0)
    disappearButton.BackgroundColor3 = SETTINGS.Background
    disappearButton.BackgroundTransparency = 0.2
    disappearButton.Text = "HIDE UI"
    disappearButton.TextColor3 = Color3.fromRGB(255, 255, 255)
    disappearButton.Font = Enum.Font.Code
    disappearButton.TextSize = 11
    
    Instance.new("UICorner", disappearButton).CornerRadius = UDim.new(0, 6)
    local masterStroke = Instance.new("UIStroke", disappearButton)
    masterStroke.Color = SETTINGS.BoxColor
    masterStroke.Thickness = 1.5

    local isUiVisible = true
    disappearButton.MouseButton1Click:Connect(function()
        isUiVisible = not isUiVisible
        buttonContainer.Visible = isUiVisible
        
        if isUiVisible then
            disappearButton.Text = "HIDE UI"
            disappearButton.TextColor3 = Color3.fromRGB(255, 255, 255)
            masterStroke.Color = SETTINGS.BoxColor
        else
            disappearButton.Text = "SHOW UI"
            disappearButton.TextColor3 = SETTINGS.BoxColor
            masterStroke.Color = Color3.fromRGB(255, 255, 255)
        end
    end)

    --// AUTO AIR TRIGGER KEY (Nested inside container)
    airButton = Instance.new("TextButton", buttonContainer)
    airButton.Size = UDim2.new(0, 70, 0, 70)
    airButton.Position = UDim2.new(0, 0, 0, 0)
    airButton.BackgroundColor3 = SETTINGS.Background
    airButton.BackgroundTransparency = 0.4
    airButton.Text = "AIR: OFF"
    airButton.TextColor3 = Color3.fromRGB(255, 255, 255)
    airButton.Font = Enum.Font.Code
    airButton.TextSize = 12
    
    Instance.new("UICorner", airButton).CornerRadius = UDim.new(0, 35)
    airButtonStroke = Instance.new("UIStroke", airButton)
    airButtonStroke.Color = SETTINGS.BoxColor
    airButtonStroke.Thickness = 2

    airButton.MouseButton1Click:Connect(function()
        toggleAutoAirState()
    end)

    --// CAMLOCK CONTROL KEY (Nested inside container)
    local lockButton = Instance.new("TextButton", buttonContainer)
    lockButton.Size = UDim2.new(0, 70, 0, 70)
    lockButton.Position = UDim2.new(0, 0, 0, 85)
    lockButton.BackgroundColor3 = SETTINGS.Background
    lockButton.BackgroundTransparency = 0.4
    lockButton.Text = "LOCK"
    lockButton.TextColor3 = Color3.fromRGB(255, 255, 255)
    lockButton.Font = Enum.Font.Code
    lockButton.TextSize = 13
    
    Instance.new("UICorner", lockButton).CornerRadius = UDim.new(0, 35)
    local stroke1 = Instance.new("UIStroke", lockButton)
    stroke1.Color = SETTINGS.BoxColor
    stroke1.Thickness = 2

    lockButton.MouseButton1Click:Connect(function()
        isLockActive = not isLockActive
        if isLockActive then
            currentTarget = getClosestPlayerToCenter()
            if currentTarget then
                lockButton.TextColor3 = SETTINGS.BoxColor
                stroke1.Thickness = 3
            else
                isLockActive = false
            end
        else
            currentTarget = nil
            lockButton.TextColor3 = Color3.fromRGB(255, 255, 255)
            stroke1.Thickness = 2
        end
    end)

    --// VISUAL ESP CONTROL KEY (Nested inside container)
    local espButton = Instance.new("TextButton", buttonContainer)
    espButton.Size = UDim2.new(0, 70, 0, 70)
    espButton.Position = UDim2.new(0, 0, 0, 170)
    espButton.BackgroundColor3 = SETTINGS.Background
    espButton.BackgroundTransparency = 0.4
    espButton.Text = "ESP: OFF"
    espButton.TextColor3 = Color3.fromRGB(255, 255, 255)
    espButton.Font = Enum.Font.Code
    espButton.TextSize = 12
    
    Instance.new("UICorner", espButton).CornerRadius = UDim.new(0, 35)
    local stroke2 = Instance.new("UIStroke", espButton)
    stroke2.Color = SETTINGS.ESPColor
    stroke2.Thickness = 2

    espButton.MouseButton1Click:Connect(function()
        local newState = not espActive
        toggleESP(newState)
        if newState then
            espButton.Text = "ESP: ON"
            espButton.TextColor3 = SETTINGS.ESPColor
            stroke2.Thickness = 3
        else
            espButton.Text = "ESP: OFF"
            espButton.TextColor3 = Color3.fromRGB(255, 255, 255)
            stroke2.Thickness = 2
        end
    end)
end

--========================================================================--
-- 8. INITIALIZATION EXECUTION
--========================================================================--
task.spawn(playIntro)
task.spawn(setupMobileUI)

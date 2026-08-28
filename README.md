local Players = game:GetService("Players")
local Workspace = game:GetService("Workspace")
local Lighting = game:GetService("Lighting")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local RbxAnalyticsService = game:GetService("RbxAnalyticsService")

local LocalPlayer = Players.LocalPlayer
local PlayerGui = LocalPlayer:WaitForChild("PlayerGui")

local function generateRandomString(length)
    local chars = "abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ"
    local res = ""
    for _ = 1, (length or math.random(8, 14)) do
        local rand = math.random(1, #chars)
        res = res .. string.sub(chars, rand, rand)
    end
    return res
end

local randomUIName = "Gui_" .. generateRandomString(12)
if PlayerGui:FindFirstChild(randomUIName) then
    PlayerGui[randomUIName]:Destroy()
end

local gui = Instance.new("ScreenGui", PlayerGui)
gui.Name = randomUIName
gui.ResetOnSpawn = false
gui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling

local COLOR_DARK = Color3.fromRGB(15, 15, 15)
local COLOR_BG = Color3.fromRGB(25, 25, 25)
local COLOR_GRAY = Color3.fromRGB(150, 150, 150)
local COLOR_BRIGHT_GRAY = Color3.fromRGB(210, 210, 210)
local COLOR_ACTIVE_GRAY = Color3.fromRGB(190, 190, 190)

local state = {
    autoSpin = false,
    espName = false,
    espFull = false,
    espTrace = false,
    espSkeleton = false,
    aimBot = false,
    aimLock = false,
    silentAim = false,
    magicBullet = false,
    noRecoil = false,
    dameAo = false,
    teleport = false,
    spd = false,
    fly = false,
    fixLag = false,
    fog = false,
    noclip = false,
    speedVal = 24,
    flyHeight = 0,
    spinSpeed = 45,
    povRadius = 120,
    isShooting = false,
    currentLockedTarget = nil,
    serverStartTime = 0,
    keyExpireTimestamp = 0,
    keyDurationText = "Permanent (Vĩnh Viễn)",
    themeMode = "RGB",
    currentThemeColor = Color3.fromRGB(150, 150, 150)
}

local function getHWID()
    local success, id = pcall(function()
        if RbxAnalyticsService and RbxAnalyticsService.GetClientId then
            return RbxAnalyticsService:GetClientId()
        end
        return "DEFAULT_HWID_DEVICE"
    end)
    if success and id then return id end
    return "DEFAULT_HWID_DEVICE"
end

local currentDeviceHWID = getHWID()

local bgMusic = Instance.new("Sound")
bgMusic.SoundId = "rbxassetid://9043887091"
bgMusic.Volume = 0.3
bgMusic.Looped = true
bgMusic.Parent = gui
bgMusic:Play()

local function playClickSound()
    local sound = Instance.new("Sound")
    sound.SoundId = "rbxassetid://6042457223"
    sound.Volume = 0.6
    sound.Parent = gui
    sound:Play()
    task.delay(1, function()
        if sound then sound:Destroy() end
    end)
end

local function formatText(label, text, textSize, textColor)
    label.Text = text
    label.TextSize = textSize or 12
    label.TextColor3 = textColor or COLOR_GRAY
    label.Font = Enum.Font.GothamBold
end

local keyFrame = Instance.new("Frame")
keyFrame.Size = UDim2.new(0, 340, 0, 210)
keyFrame.Position = UDim2.new(0.5, -170, 0.5, -105)
keyFrame.BackgroundColor3 = COLOR_DARK
keyFrame.BackgroundTransparency = 0.15
keyFrame.Active = true
keyFrame.Draggable = true
keyFrame.Parent = gui
Instance.new("UICorner", keyFrame).CornerRadius = UDim.new(0, 14)

local keyStroke = Instance.new("UIStroke", keyFrame)
keyStroke.Thickness = 2.5
keyStroke.Color = COLOR_GRAY

task.spawn(function()
    while keyFrame and keyFrame.Parent do
        for i = 0, 1, 0.005 do
            if state.themeMode == "RGB" then
                local col = Color3.fromHSV(i, 0.9, 1)
                keyStroke.Color = col
                state.currentThemeColor = col
            end
            task.wait(0.04)
        end
    end
end)

local titleLabel = Instance.new("TextLabel")
titleLabel.Size = UDim2.new(1, 0, 0, 35)
titleLabel.Position = UDim2.new(0, 0, 0, 5)
titleLabel.BackgroundTransparency = 1
titleLabel.Parent = keyFrame
formatText(titleLabel, "Menu Dnqkhdz V4", 16, COLOR_GRAY)
titleLabel.Font = Enum.Font.GothamBlack

local keyInput = Instance.new("TextBox")
keyInput.Size = UDim2.new(0, 240, 0, 35)
keyInput.Position = UDim2.new(0.5, -135, 0, 45)
keyInput.BackgroundColor3 = COLOR_BG
keyInput.BackgroundTransparency = 0.2
keyInput.PlaceholderText = "Nhap Key vao day..."
keyInput.Parent = keyFrame
Instance.new("UICorner", keyInput).CornerRadius = UDim.new(0, 8)
formatText(keyInput, "", 13, COLOR_GRAY)

local statusLight = Instance.new("Frame")
statusLight.Size = UDim2.new(0, 30, 0, 35)
statusLight.Position = UDim2.new(0.5, 105, 0, 45)
statusLight.BackgroundColor3 = Color3.fromRGB(80, 80, 80)
statusLight.BackgroundTransparency = 0.2
statusLight.Parent = keyFrame
Instance.new("UICorner", statusLight).CornerRadius = UDim.new(0, 8)

local statusLightStroke = Instance.new("UIStroke", statusLight)
statusLightStroke.Color = Color3.fromRGB(0, 0, 0)
statusLightStroke.Thickness = 1.5

local keyStatusLabel = Instance.new("TextLabel")
keyStatusLabel.Size = UDim2.new(1, 0, 0, 30)
keyStatusLabel.Position = UDim2.new(0, 0, 0, 90)
keyStatusLabel.BackgroundTransparency = 1
keyStatusLabel.Parent = keyFrame
formatText(keyStatusLabel, "Vui long nhap Key hop le de tiep tuc", 11, COLOR_GRAY)

local submitKeyBtn = Instance.new("TextButton")
submitKeyBtn.Size = UDim2.new(0, 290, 0, 38)
submitKeyBtn.Position = UDim2.new(0.5, -145, 0, 135)
submitKeyBtn.BackgroundColor3 = COLOR_BG
submitKeyBtn.BackgroundTransparency = 0.2
submitKeyBtn.Parent = keyFrame
Instance.new("UICorner", submitKeyBtn).CornerRadius = UDim.new(0, 8)
formatText(submitKeyBtn, "XAC NHAN KEY", 12, COLOR_BRIGHT_GRAY)

local topContainer = Instance.new("Frame")
topContainer.Size = UDim2.new(0, 300, 0, 75)
topContainer.Position = UDim2.new(0.5, -150, 0, 10)
topContainer.BackgroundTransparency = 1
topContainer.Visible = false
topContainer.Parent = gui

local headerTitle = Instance.new("TextLabel")
headerTitle.Size = UDim2.new(1, 0, 0, 28)
headerTitle.Position = UDim2.new(0, 0, 0, 0)
headerTitle.BackgroundTransparency = 1
headerTitle.Parent = topContainer
formatText(headerTitle, "Menu Dnqkhdz V4", 17, COLOR_GRAY)
headerTitle.Font = Enum.Font.GothamBlack
local headerTitleStroke = Instance.new("UIStroke", headerTitle)
headerTitleStroke.Color = Color3.fromRGB(0, 0, 0)
headerTitleStroke.Thickness = 3

local keyExpiryLabel = Instance.new("TextLabel")
keyExpiryLabel.Size = UDim2.new(1, 0, 0, 20)
keyExpiryLabel.Position = UDim2.new(0, 0, 0, 28)
keyExpiryLabel.BackgroundTransparency = 1
keyExpiryLabel.Parent = topContainer
formatText(keyExpiryLabel, "Expiry: Permanent", 11, Color3.fromRGB(80, 255, 80))
local expiryStroke = Instance.new("UIStroke", keyExpiryLabel)
expiryStroke.Color = Color3.fromRGB(0, 0, 0)
expiryStroke.Thickness = 2

local fpsLabel = Instance.new("TextLabel")
fpsLabel.Size = UDim2.new(1, 0, 0, 20)
fpsLabel.Position = UDim2.new(0, 0, 0, 50)
fpsLabel.BackgroundTransparency = 1
fpsLabel.Parent = topContainer
formatText(fpsLabel, "FPS: 60", 14, COLOR_BRIGHT_GRAY)
local fpsStroke = Instance.new("UIStroke", fpsLabel)
fpsStroke.Color = Color3.fromRGB(0, 0, 0)
fpsStroke.Thickness = 2

local povCircle = Instance.new("Frame")
povCircle.Size = UDim2.new(0, state.povRadius * 2, 0, state.povRadius * 2)
povCircle.AnchorPoint = Vector2.new(0.5, 0.5)
povCircle.Position = UDim2.new(0.5, 0, 0.5, 0)
povCircle.BackgroundTransparency = 1
povCircle.Visible = false
povCircle.Parent = gui
Instance.new("UICorner", povCircle).CornerRadius = UDim.new(1, 0)
local povStroke = Instance.new("UIStroke", povCircle)
povStroke.Thickness = 2.5
povStroke.Color = Color3.fromRGB(100, 100, 100)

local mainWindow = Instance.new("Frame")
mainWindow.Size = UDim2.new(0, 460, 0, 310)
mainWindow.Position = UDim2.new(0.5, -230, 0.5, -155)
mainWindow.BackgroundColor3 = COLOR_DARK
mainWindow.BackgroundTransparency = 0.12
mainWindow.Active = true
mainWindow.Draggable = true
mainWindow.Visible = false
mainWindow.Parent = gui
Instance.new("UICorner", mainWindow).CornerRadius = UDim.new(0, 14)

local mainStroke = Instance.new("UIStroke", mainWindow)
mainStroke.Thickness = 2.5

task.spawn(function()
    while true do
        for i = 0, 1, 0.005 do
            if state.themeMode == "RGB" then
                local col = Color3.fromHSV(i, 0.9, 1)
                mainStroke.Color = col
            end
            task.wait(0.04)
        end
    end
end)

local bgChillImg = Instance.new("ImageLabel")
bgChillImg.Size = UDim2.new(0, 180, 0, 180)
bgChillImg.Position = UDim2.new(1, -190, 1, -190)
bgChillImg.BackgroundTransparency = 1
bgChillImg.Image = "rbxassetid://13838274026"
bgChillImg.ImageTransparency = 0.65
bgChillImg.Parent = mainWindow

local openIconButton = Instance.new("TextButton")
openIconButton.Size = UDim2.new(0, 50, 0, 50)
openIconButton.Position = UDim2.new(0.05, 0, 0.2, 0)
openIconButton.BackgroundColor3 = Color3.fromRGB(20, 20, 20)
openIconButton.BackgroundTransparency = 0.15
openIconButton.Text = "K"
openIconButton.TextColor3 = Color3.fromRGB(255, 255, 255)
openIconButton.TextSize = 24
openIconButton.Font = Enum.Font.GothamBlack
openIconButton.Active = true
openIconButton.Draggable = true
openIconButton.Visible = false
openIconButton.Parent = gui
Instance.new("UICorner", openIconButton).CornerRadius = UDim.new(0, 12)

local iconStroke = Instance.new("UIStroke", openIconButton)
iconStroke.Thickness = 2.5
iconStroke.Color = Color3.fromRGB(255, 255, 255)

task.spawn(function()
    while openIconButton and openIconButton.Parent do
        for i = 0, 1, 0.005 do
            local col = Color3.fromHSV(i, 0.9, 1)
            iconStroke.Color = col
            openIconButton.TextColor3 = col
            task.wait(0.04)
        end
    end
end)

local closeBtn = Instance.new("TextButton")
closeBtn.Size = UDim2.new(0, 26, 0, 26)
closeBtn.Position = UDim2.new(1, -32, 0, 6)
closeBtn.BackgroundColor3 = COLOR_BG
closeBtn.BackgroundTransparency = 0.2
closeBtn.Parent = mainWindow
Instance.new("UICorner", closeBtn).CornerRadius = UDim.new(0, 6)
formatText(closeBtn, "-", 18, COLOR_GRAY)

local menuFullTitle = Instance.new("TextLabel")
menuFullTitle.Size = UDim2.new(0, 220, 0, 35)
menuFullTitle.Position = UDim2.new(0, 130, 0, 6)
menuFullTitle.BackgroundTransparency = 1
menuFullTitle.Parent = mainWindow
formatText(menuFullTitle, "Menu Dnqkhdz V4", 15, COLOR_GRAY)
menuFullTitle.Font = Enum.Font.GothamBlack
local menuTitleStroke = Instance.new("UIStroke", menuFullTitle)
menuTitleStroke.Color = Color3.fromRGB(0, 0, 0)
menuTitleStroke.Thickness = 2.5

local searchBar = Instance.new("TextBox")
searchBar.Size = UDim2.new(0, 100, 0, 30)
searchBar.Position = UDim2.new(1, -145, 0, 8)
searchBar.BackgroundColor3 = COLOR_BG
searchBar.BackgroundTransparency = 0.2
searchBar.PlaceholderText = "Search..."
searchBar.Parent = mainWindow
Instance.new("UICorner", searchBar).CornerRadius = UDim.new(0, 6)
formatText(searchBar, "", 10, COLOR_GRAY)

local function createTabBtn(posY, name)
    local btn = Instance.new("TextButton")
    btn.Size = UDim2.new(0, 110, 0, 28)
    btn.Position = UDim2.new(0, 10, 0, posY)
    btn.BackgroundColor3 = COLOR_BG
    btn.BackgroundTransparency = 0.2
    btn.Parent = mainWindow
    Instance.new("UICorner", btn).CornerRadius = UDim.new(0, 8)
    formatText(btn, name, 11, COLOR_GRAY)
    return btn
end

local tProfile = createTabBtn(45, "Profile")
local tTime = createTabBtn(80, "Time")
local t1 = createTabBtn(115, "Esp")
local t2 = createTabBtn(150, "Aim & Combat")
local t3 = createTabBtn(185, "Setting")

local function createPanel(canvasHeight)
    local panel = Instance.new("ScrollingFrame")
    panel.Size = UDim2.new(0, 320, 0, 215)
    panel.Position = UDim2.new(0, 130, 0, 45)
    panel.BackgroundTransparency = 1
    panel.BorderSizePixel = 0
    panel.CanvasSize = UDim2.new(0, 0, 0, canvasHeight)
    panel.ScrollBarThickness = 4
    panel.ScrollingEnabled = true
    panel.Visible = false
    panel.Parent = mainWindow
    return panel
end

local pProfile = createPanel(230)
pProfile.Visible = true
local pTime = createPanel(230)
local p1 = createPanel(290)
local p2 = createPanel(350)
local p3 = createPanel(230)

local avatarImg = Instance.new("ImageLabel")
avatarImg.Size = UDim2.new(0, 50, 0, 50)
avatarImg.Position = UDim2.new(0, 10, 0, 10)
avatarImg.BackgroundColor3 = COLOR_BG
avatarImg.BackgroundTransparency = 0.2
avatarImg.Parent = pProfile
Instance.new("UICorner", avatarImg).CornerRadius = UDim.new(0, 8)
pcall(function()
    avatarImg.Image = Players:GetUserThumbnailAsync(LocalPlayer.UserId, Enum.ThumbnailType.HeadShot, Enum.ThumbnailSize.Size150x150)
end)

local profileNameLab = Instance.new("TextLabel")
profileNameLab.Size = UDim2.new(0, 240, 0, 20)
profileNameLab.Position = UDim2.new(0, 70, 0, 10)
profileNameLab.BackgroundTransparency = 1
profileNameLab.Parent = pProfile
formatText(profileNameLab, "Name: " .. LocalPlayer.Name, 12, COLOR_BRIGHT_GRAY)
profileNameLab.TextXAlignment = Enum.TextXAlignment.Left

local profilePingLab = Instance.new("TextLabel")
profilePingLab.Size = UDim2.new(0, 240, 0, 20)
profilePingLab.Position = UDim2.new(0, 70, 0, 35)
profilePingLab.BackgroundTransparency = 1
profilePingLab.Parent = pProfile
formatText(profilePingLab, "Ping: Calculating...", 11, COLOR_GRAY)
profilePingLab.TextXAlignment = Enum.TextXAlignment.Left

local profileKeyBox = Instance.new("TextBox")
profileKeyBox.Size = UDim2.new(1, 0, 0, 35)
profileKeyBox.Position = UDim2.new(0, 0, 0, 75)
profileKeyBox.BackgroundColor3 = COLOR_BG
profileKeyBox.BackgroundTransparency = 0.2
profileKeyBox.TextEditable = false
profileKeyBox.Parent = pProfile
Instance.new("UICorner", profileKeyBox).CornerRadius = UDim.new(0, 6)
formatText(profileKeyBox, "Key Status: Permanent (Vinh Vien)", 11, Color3.fromRGB(80, 255, 80))

local profileHWIDBox = Instance.new("TextBox")
profileHWIDBox.Size = UDim2.new(1, 0, 0, 35)
profileHWIDBox.Position = UDim2.new(0, 0, 0, 120)
profileHWIDBox.BackgroundColor3 = COLOR_BG
profileHWIDBox.BackgroundTransparency = 0.2
profileHWIDBox.TextEditable = false
profileHWIDBox.Parent = pProfile
Instance.new("UICorner", profileHWIDBox).CornerRadius = UDim.new(0, 6)
formatText(profileHWIDBox, "HWID: " .. string.sub(currentDeviceHWID, 1, 24) .. "...", 10, COLOR_GRAY)

local profileInfoNote = Instance.new("TextLabel")
profileInfoNote.Size = UDim2.new(1, 0, 0, 50)
profileInfoNote.Position = UDim2.new(0, 0, 0, 165)
profileInfoNote.BackgroundTransparency = 1
profileInfoNote.TextWrapped = true
profileInfoNote.Parent = pProfile
formatText(profileInfoNote, "Dkhoi Cheat Rbl - Gunfight Script V4", 10, Color3.fromRGB(100, 100, 100))

local lblKeyTimer = Instance.new("TextLabel")
lblKeyTimer.Size = UDim2.new(1, 0, 0, 45)
lblKeyTimer.Position = UDim2.new(0, 0, 0, 10)
lblKeyTimer.BackgroundColor3 = COLOR_BG
lblKeyTimer.BackgroundTransparency = 0.2
lblKeyTimer.Parent = pTime
Instance.new("UICorner", lblKeyTimer).CornerRadius = UDim.new(0, 6)
formatText(lblKeyTimer, "Key Time Remaining: Permanent (Vĩnh Viễn)", 11, COLOR_BRIGHT_GRAY)

local lblServerTimer = Instance.new("TextLabel")
lblServerTimer.Size = UDim2.new(1, 0, 0, 45)
lblServerTimer.Position = UDim2.new(0, 0, 0, 65)
lblServerTimer.BackgroundColor3 = COLOR_BG
lblServerTimer.BackgroundTransparency = 0.2
lblServerTimer.Parent = pTime
Instance.new("UICorner", lblServerTimer).CornerRadius = UDim.new(0, 6)
formatText(lblServerTimer, "Time in Server: 00:00:00", 11, COLOR_BRIGHT_GRAY)

local function setToggleState(btnFrame, active)
    local dot = btnFrame:FindFirstChild("Dot")
    local stroke = btnFrame:FindFirstChildOfClass("UIStroke")
    if active then
        btnFrame.BackgroundColor3 = COLOR_ACTIVE_GRAY
        dot.Position = UDim2.new(1, -21, 0.5, -9)
        dot.BackgroundColor3 = COLOR_DARK
        if stroke then stroke.Color = Color3.fromRGB(255, 255, 255) end
    else
        btnFrame.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
        dot.Position = UDim2.new(0, 3, 0.5, -9)
        dot.BackgroundColor3 = Color3.fromRGB(180, 180, 180)
        if stroke then stroke.Color = Color3.fromRGB(80, 80, 80) end
    end
end

local function addToggleComponent(parentPanel, labelText, posY)
    local frame = Instance.new("Frame")
    frame.Size = UDim2.new(1, 0, 0, 30)
    frame.Position = UDim2.new(0, 0, 0, posY)
    frame.BackgroundColor3 = COLOR_BG
    frame.BackgroundTransparency = 0.2
    frame.Parent = parentPanel
    Instance.new("UICorner", frame).CornerRadius = UDim.new(0, 6)

    local frameStroke = Instance.new("UIStroke", frame)
    frameStroke.Color = Color3.fromRGB(90, 180, 255)
    frameStroke.Thickness = 1.2

    local label = Instance.new("TextLabel")
    label.Size = UDim2.new(0, 180, 1, 0)
    label.Position = UDim2.new(0, 10, 0, 0)
    label.BackgroundTransparency = 1
    label.TextXAlignment = Enum.TextXAlignment.Left
    label.Parent = frame
    formatText(label, labelText, 11, Color3.new(1, 1, 1))

    local sw = Instance.new("TextButton")
    sw.Size = UDim2.new(0, 48, 0, 20)
    sw.Position = UDim2.new(1, -55, 0.5, -10)
    sw.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
    sw.BackgroundTransparency = 0.2
    sw.Text = ""
    sw.Parent = frame
    Instance.new("UICorner", sw).CornerRadius = UDim.new(0, 11)

    local swStroke = Instance.new("UIStroke", sw)
    swStroke.Thickness = 1.5
    swStroke.Color = Color3.fromRGB(80, 80, 80)

    local dot = Instance.new("Frame")
    dot.Size = UDim2.new(0, 16, 0, 16)
    dot.Position = UDim2.new(0, 2, 0.5, -8)
    dot.BackgroundColor3 = Color3.fromRGB(180, 180, 180)
    dot.Name = "Dot"
    dot.Parent = sw
    Instance.new("UICorner", dot).CornerRadius = UDim.new(1, 0)

    return sw, frame
end

local function addRowComponent(parentPanel, labelText, posY, opt1, opt2, opt3)
    local frame = Instance.new("Frame")
    frame.Size = UDim2.new(1, 0, 0, 44)
    frame.Position = UDim2.new(0, 0, 0, posY)
    frame.BackgroundColor3 = COLOR_BG
    frame.BackgroundTransparency = 0.2
    frame.Parent = parentPanel
    Instance.new("UICorner", frame).CornerRadius = UDim.new(0, 6)

    local frameStroke = Instance.new("UIStroke", frame)
    frameStroke.Color = Color3.fromRGB(90, 180, 255)
    frameStroke.Thickness = 1.2

    local label = Instance.new("TextLabel")
    label.Size = UDim2.new(0, 150, 0, 20)
    label.Position = UDim2.new(0, 10, 0, 2)
    label.BackgroundTransparency = 1
    label.TextXAlignment = Enum.TextXAlignment.Left
    label.Parent = frame
    formatText(label, labelText, 11, Color3.new(1, 1, 1))

    local sw = Instance.new("TextButton")
    sw.Size = UDim2.new(0, 48, 0, 18)
    sw.Position = UDim2.new(1, -55, 0, 2)
    sw.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
    sw.BackgroundTransparency = 0.2
    sw.Text = ""
    sw.Parent = frame
    Instance.new("UICorner", sw).CornerRadius = UDim.new(0, 10)

    local swStroke = Instance.new("UIStroke", sw)
    swStroke.Thickness = 1.5
    swStroke.Color = Color3.fromRGB(80, 80, 80)

    local dot = Instance.new("Frame")
    dot.Size = UDim2.new(0, 14, 0, 14)
    dot.Position = UDim2.new(0, 2, 0.5, -7)
    dot.BackgroundColor3 = Color3.fromRGB(180, 180, 180)
    dot.Name = "Dot"
    dot.Parent = sw
    Instance.new("UICorner", dot).CornerRadius = UDim.new(1, 0)

    local function createBtnOption(text, posX)
        local btn = Instance.new("TextButton")
        btn.Size = UDim2.new(0, 88, 0, 20)
        btn.Position = UDim2.new(0, posX, 0, 20)
        btn.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
        btn.BackgroundTransparency = 0.2
        btn.Parent = frame
        Instance.new("UICorner", btn).CornerRadius = UDim.new(0, 4)
        formatText(btn, text, 9)
        return btn
    end

    local b1 = createBtnOption(opt1, 6)
    local b2 = createBtnOption(opt2, 100)
    local b3 = createBtnOption(opt3, 194)

    local function selector(chosen)
        for _, b in ipairs({b1, b2, b3}) do
            b.BackgroundColor3 = (b == chosen) and COLOR_GRAY or Color3.fromRGB(50, 50, 50)
            b.TextColor3 = (b == chosen) and COLOR_DARK or Color3.new(1, 1, 1)
        end
    end
    selector(b1)
    return sw, b1, b2, b3, selector, frame
end

local espNameB, fEspName = addToggleComponent(p1, "ESP Name", 0)
local espFullB, fEspFull = addToggleComponent(p1, "ESP Full", 35)
local espTraceB, fEspTrace = addToggleComponent(p1, "ESP Traces", 70)
local espSkeletonB, fEspSkeleton = addToggleComponent(p1, "ESP Skeleton", 105)
local noclipB, fNoclip = addToggleComponent(p1, "Noclip", 140)
local teleportB, fTp = addToggleComponent(p1, "Teleport to Target", 175)
local spdB, s1, s2, s3, selS, fSpd = addRowComponent(p1, "SpeedRun", 210, "Normal", "Fast", "Super")
local flyB, f1, f2, f3, selF, fFly = addRowComponent(p1, "Fly", 260, "Low", "Medium", "High")

local aimBotB, fAimBot = addToggleComponent(p2, "AimBot", 0)
local aimLockB, fAimLock = addToggleComponent(p2, "AimLock", 35)
local silentAimB, fSilentAim = addToggleComponent(p2, "Silent Aim", 70)
local magicBulletB, fMagicBullet = addToggleComponent(p2, "Magic Bullet", 105)
local noRecoilB, fNoRecoil = addToggleComponent(p2, "No Recoil", 140)
local spinB, fSpin = addToggleComponent(p2, "SpinBot", 175)
local dameAoB, fDameAo = addToggleComponent(p2, "Fake Damage", 210)

local fixLagB, fFixLag = addToggleComponent(p3, "Fix Lag", 0)
local fogB, fFog = addToggleComponent(p3, "Atmospheric Fog", 36)

local function createThemeBtn(posY, name)
    local btn = Instance.new("TextButton")
    btn.Size = UDim2.new(1, 0, 0, 32)
    btn.Position = UDim2.new(0, 0, 0, posY)
    btn.BackgroundColor3 = COLOR_BG
    btn.BackgroundTransparency = 0.2
    btn.Parent = p3
    Instance.new("UICorner", btn).CornerRadius = UDim.new(0, 6)
    
    local btnStroke = Instance.new("UIStroke", btn)
    btnStroke.Color = Color3.fromRGB(90, 180, 255)
    btnStroke.Thickness = 1.2
    
    formatText(btn, name, 11, COLOR_BRIGHT_GRAY)
    return btn
end

local themeBtn1 = createThemeBtn(80, "Theme: RGB Animation")
local themeBtn2 = createThemeBtn(120, "Theme: Neon Green")
local themeBtn3 = createThemeBtn(160, "Theme: Cyberpunk Red")

submitKeyBtn.MouseButton1Click:Connect(function()
    playClickSound()
    local inputKey = keyInput.Text
    local allowLogin = false

    if inputKey == "dangkhoilatui" then
        allowLogin = true
    end

    if allowLogin then
        statusLight.BackgroundColor3 = Color3.fromRGB(0, 255, 0)
        formatText(keyStatusLabel, "Thanh cong! Dang vao Menu...", 11, COLOR_BRIGHT_GRAY)
        state.serverStartTime = os.time()
        
        task.wait(0.6)
        keyFrame:Destroy()
        mainWindow.Visible = true
        topContainer.Visible = true
        keyExpiryLabel.Text = "Expiry: Permanent"
    else
        statusLight.BackgroundColor3 = Color3.fromRGB(255, 0, 0)
        formatText(keyStatusLabel, "Key khong chinh xac!", 11, Color3.fromRGB(200, 60, 60))
    end
end)

closeBtn.MouseButton1Click:Connect(function() 
    playClickSound() 
    mainWindow.Visible = false 
    bgChillImg.Visible = false
    openIconButton.Visible = true 
end)

openIconButton.MouseButton1Click:Connect(function() 
    playClickSound() 
    mainWindow.Visible = true 
    bgChillImg.Visible = true
    openIconButton.Visible = false 
end)

local function switchTab(tabToShow)
    pProfile.Visible = (tabToShow == pProfile)
    pTime.Visible = (tabToShow == pTime)
    p1.Visible = (tabToShow == p1)
    p2.Visible = (tabToShow == p2)
    p3.Visible = (tabToShow == p3)
    
    for _, btn in ipairs({tProfile, tTime, t1, t2, t3}) do
        btn.BackgroundColor3 = COLOR_BG
        btn.TextColor3 = COLOR_GRAY
    end
end

tProfile.MouseButton1Click:Connect(function() playClickSound(); switchTab(pProfile); tProfile.BackgroundColor3 = COLOR_GRAY; tProfile.TextColor3 = COLOR_DARK end)
tTime.MouseButton1Click:Connect(function() playClickSound(); switchTab(pTime); tTime.BackgroundColor3 = COLOR_GRAY; tTime.TextColor3 = COLOR_DARK end)
t1.MouseButton1Click:Connect(function() playClickSound(); switchTab(p1); t1.BackgroundColor3 = COLOR_GRAY; t1.TextColor3 = COLOR_DARK end)
t2.MouseButton1Click:Connect(function() playClickSound(); switchTab(p2); t2.BackgroundColor3 = COLOR_GRAY; t2.TextColor3 = COLOR_DARK end)
t3.MouseButton1Click:Connect(function() playClickSound(); switchTab(p3); t3.BackgroundColor3 = COLOR_GRAY; t3.TextColor3 = COLOR_DARK end)

tProfile.BackgroundColor3 = COLOR_GRAY; tProfile.TextColor3 = COLOR_DARK

espNameB.MouseButton1Click:Connect(function() playClickSound(); state.espName = not state.espName; setToggleState(espNameB, state.espName) end)
espFullB.MouseButton1Click:Connect(function() playClickSound(); state.espFull = not state.espFull; setToggleState(espFullB, state.espFull) end)
espTraceB.MouseButton1Click:Connect(function() playClickSound(); state.espTrace = not state.espTrace; setToggleState(espTraceB, state.espTrace) end)
espSkeletonB.MouseButton1Click:Connect(function() playClickSound(); state.espSkeleton = not state.espSkeleton; setToggleState(espSkeletonB, state.espSkeleton) end)
noclipB.MouseButton1Click:Connect(function() playClickSound(); state.noclip = not state.noclip; setToggleState(noclipB, state.noclip) end)
teleportB.MouseButton1Click:Connect(function() playClickSound(); state.teleport = not state.teleport; setToggleState(teleportB, state.teleport) end)
spdB.MouseButton1Click:Connect(function() playClickSound(); state.spd = not state.spd; setToggleState(spdB, state.spd) end)
flyB.MouseButton1Click:Connect(function() playClickSound(); state.fly = not state.fly; setToggleState(flyB, state.fly) end)

aimBotB.MouseButton1Click:Connect(function() 
    playClickSound()
    state.aimBot = not state.aimBot
    setToggleState(aimBotB, state.aimBot)
    povCircle.Visible = state.aimBot or state.aimLock or state.silentAim
end)

aimLockB.MouseButton1Click:Connect(function() 
    playClickSound()
    state.aimLock = not state.aimLock
    setToggleState(aimLockB, state.aimLock)
    povCircle.Visible = state.aimBot or state.aimLock or state.silentAim
    state.currentLockedTarget = nil
end)

silentAimB.MouseButton1Click:Connect(function()
    playClickSound()
    state.silentAim = not state.silentAim
    setToggleState(silentAimB, state.silentAim)
    povCircle.Visible = state.aimBot or state.aimLock or state.silentAim
end)

magicBulletB.MouseButton1Click:Connect(function() playClickSound(); state.magicBullet = not state.magicBullet; setToggleState(magicBulletB, state.magicBullet) end)
noRecoilB.MouseButton1Click:Connect(function() playClickSound(); state.noRecoil = not state.noRecoil; setToggleState(noRecoilB, state.noRecoil) end)
spinB.MouseButton1Click:Connect(function() playClickSound(); state.autoSpin = not state.autoSpin; setToggleState(spinB, state.autoSpin) end)
dameAoB.MouseButton1Click:Connect(function() playClickSound(); state.dameAo = not state.dameAo; setToggleState(dameAoB, state.dameAo) end)

themeBtn1.MouseButton1Click:Connect(function()
    playClickSound()
    state.themeMode = "RGB"
    formatText(themeBtn1, "Theme: RGB Animation (Active)", 11, Color3.fromRGB(80, 255, 80))
    formatText(themeBtn2, "Theme: Neon Green", 11, COLOR_BRIGHT_GRAY)
    formatText(themeBtn3, "Theme: Cyberpunk Red", 11, COLOR_BRIGHT_GRAY)
end)

themeBtn2.MouseButton1Click:Connect(function()
    playClickSound()
    state.themeMode = "NeonGreen"
    state.currentThemeColor = Color3.fromRGB(0, 255, 100)
    mainStroke.Color = state.currentThemeColor
    keyStroke.Color = state.currentThemeColor
    headerTitle.TextColor3 = state.currentThemeColor
    menuFullTitle.TextColor3 = state.currentThemeColor
    iconStroke.Color = state.currentThemeColor
    openIconButton.TextColor3 = state.currentThemeColor
    formatText(themeBtn1, "Theme: RGB Animation", 11, COLOR_BRIGHT_GRAY)
    formatText(themeBtn2, "Theme: Neon Green (Active)", 11, Color3.fromRGB(80, 255, 80))
    formatText(themeBtn3, "Theme: Cyberpunk Red", 11, COLOR_BRIGHT_GRAY)
end)

themeBtn3.MouseButton1Click:Connect(function()
    playClickSound()
    state.themeMode = "CyberpunkRed"
    state.currentThemeColor = Color3.fromRGB(255, 40, 60)
    mainStroke.Color = state.currentThemeColor
    keyStroke.Color = state.currentThemeColor
    headerTitle.TextColor3 = state.currentThemeColor
    menuFullTitle.TextColor3 = state.currentThemeColor
    iconStroke.Color = state.currentThemeColor
    openIconButton.TextColor3 = state.currentThemeColor
    formatText(themeBtn1, "Theme: RGB Animation", 11, COLOR_BRIGHT_GRAY)
    formatText(themeBtn2, "Theme: Neon Green", 11, COLOR_BRIGHT_GRAY)
    formatText(themeBtn3, "Theme: Cyberpunk Red (Active)", 11, Color3.fromRGB(80, 255, 80))
end)

s1.MouseButton1Click:Connect(function() playClickSound(); selS(s1); state.speedVal = 24 end)
s2.MouseButton1Click:Connect(function() playClickSound(); selS(s2); state.speedVal = 38 end)
s3.MouseButton1Click:Connect(function() playClickSound(); selS(s3); state.speedVal = 55 end)

f1.MouseButton1Click:Connect(function() playClickSound(); selF(f1); state.flyHeight = 15 end)
f2.MouseButton1Click:Connect(function() playClickSound(); selF(f2); state.flyHeight = 35 end)
f3.MouseButton1Click:Connect(function() playClickSound(); selF(f3); state.flyHeight = 65 end)

UserInputService.InputBegan:Connect(function(input, gpe)
    if gpe then return end
    if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
        state.isShooting = true
    end
end)

UserInputService.InputEnded:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
        state.isShooting = false
        state.currentLockedTarget = nil
    end
end)

local lastTime = os.clock()
local frameCount = 0

RunService.RenderStepped:Connect(function()
    frameCount = frameCount + 1
    local currentTime = os.clock()
    if currentTime - lastTime >= 1 then
        local calculatedFps = math.floor(frameCount / (currentTime - lastTime))
        fpsLabel.Text = "FPS: " .. calculatedFps
        frameCount = 0
        lastTime = currentTime
        pcall(function()
            if LocalPlayer.GetNetworkPing then
                local ping = math.floor(LocalPlayer:GetNetworkPing() * 1000)
                profilePingLab.Text = "Ping: " .. ping .. " ms"
            end
        end)
    end
    if state.serverStartTime > 0 then
        local elapsedServer = os.time() - state.serverStartTime
        local sHours = math.floor(elapsedServer / 3600)
        local sMins = math.floor((elapsedServer % 3600) / 60)
        local sSecs = elapsedServer % 60
        lblServerTimer.Text = string.format("Time in Server: %02d:%02d:%02d", sHours, sMins, sSecs)
    end
end)

local activeESPName = {}

RunService.RenderStepped:Connect(function()
    local rainbowColor = Color3.fromHSV((os.clock() % 5) / 5, 1, 1)

    if state.themeMode == "RGB" then
        iconStroke.Color = rainbowColor
        openIconButton.TextColor3 = rainbowColor
    end

    if state.autoSpin and LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart") then
        local hrp = LocalPlayer.Character.HumanoidRootPart
        hrp.CFrame = hrp.CFrame * CFrame.Angles(0, math.rad(state.spinSpeed), 0)
    end
    
    if state.spd and LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("Humanoid") then
        LocalPlayer.Character.Humanoid.WalkSpeed = state.speedVal
    end
    
    if state.fly and LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart") then
        local hrp = LocalPlayer.Character.HumanoidRootPart
        hrp.Velocity = Vector3.new(hrp.Velocity.X, state.flyHeight, hrp.Velocity.Z)
    end

    if state.noclip and LocalPlayer.Character then
        for _, part in ipairs(LocalPlayer.Character:GetDescendants()) do
            if part:IsA("BasePart") and part.CanCollide then
                part.CanCollide = false
            end
        end
    end
    
    for _, p in ipairs(Players:GetPlayers()) do
        if p ~= LocalPlayer then
            local char = p.Character
            local head = char and char:FindFirstChild("Head")
            local hrp = char and char:FindFirstChild("HumanoidRootPart")
            
            if char and head and hrp then
                local distVal = 0
                if LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart") then
                    distVal = math.floor((LocalPlayer.Character.HumanoidRootPart.Position - hrp.Position).Magnitude)
                end
                
                if state.espName then
                    if not activeESPName[p] then
                        local bb = Instance.new("BillboardGui")
                        bb.Name = "DnqkHdZV4NameESP"
                        bb.Adornee = head
                        bb.Size = UDim2.new(0, 120, 0, 40)
                        bb.StudsOffset = Vector3.new(0, 2.4, 0)
                        bb.AlwaysOnTop = true
                        
                        local nameLab = Instance.new("TextLabel", bb)
                        nameLab.Size = UDim2.new(1, 0, 1, 0)
                        nameLab.BackgroundTransparency = 1
                        formatText(nameLab, p.Name .. " [" .. distVal .. "m]", 12, Color3.fromRGB(255, 255, 255))
                        nameLab.Font = Enum.Font.GothamBlack
                        
                        local s1 = Instance.new("UIStroke", nameLab)
                        s1.Color = Color3.fromRGB(0, 0, 0)
                        s1.Thickness = 2
                        
                        bb.Parent = gui
                        activeESPName[p] = {gui = bb, label = nameLab}
                    else
                        activeESPName[p].gui.Adornee = head
                        activeESPName[p].label.Text = p.Name .. " [" .. distVal .. "m]"
                    end
                elseif activeESPName[p] then
                    activeESPName[p].gui:Destroy()
                    activeESPName[p] = nil
                end
            else
                if activeESPName[p] then activeESPName[p].gui:Destroy(); activeESPName[p] = nil end
            end
        end
    end
end)

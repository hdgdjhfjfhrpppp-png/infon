-- LocalScript داخل StarterPlayerScripts

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local player = Players.LocalPlayer
local playerGui = player:WaitForChild("PlayerGui")

-- GUI Container
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "FEInfoGUI"
ScreenGui.Parent = playerGui

local mainFrame = Instance.new("Frame")
mainFrame.Size = UDim2.new(0, 420, 0, 320)
mainFrame.Position = UDim2.new(0.5, -210, 0.5, -160)
mainFrame.BackgroundColor3 = Color3.fromRGB(28,28,28)
mainFrame.Visible = false
mainFrame.AnchorPoint = Vector2.new(0.5,0.5)
mainFrame.BorderSizePixel = 0
mainFrame.Parent = ScreenGui

-- Draggable
mainFrame.Active = true
mainFrame.Draggable = true

-- Title
local title = Instance.new("TextLabel")
title.Size = UDim2.new(1,0,0,40)
title.BackgroundTransparency = 1
title.Text = "معلومات اللاعب"
title.TextColor3 = Color3.fromRGB(255,255,255)
title.Font = Enum.Font.GothamBold
title.TextSize = 24
title.Parent = mainFrame

-- Player Thumbnail
local thumbnail = Instance.new("ImageLabel")
thumbnail.Size = UDim2.new(0,100,0,100)
thumbnail.Position = UDim2.new(0,10,0,50)
thumbnail.BackgroundColor3 = Color3.fromRGB(50,50,50)
thumbnail.BorderSizePixel = 0
thumbnail.Parent = mainFrame

-- Player Name
local nameLabel = Instance.new("TextLabel")
nameLabel.Size = UDim2.new(0, 250, 0, 30)
nameLabel.Position = UDim2.new(0, 120, 0, 50)
nameLabel.BackgroundTransparency = 1
nameLabel.TextColor3 = Color3.fromRGB(255,255,255)
nameLabel.Font = Enum.Font.GothamBold
nameLabel.TextSize = 20
nameLabel.TextXAlignment = Enum.TextXAlignment.Left
nameLabel.Parent = mainFrame

-- Player ID
local idLabel = Instance.new("TextLabel")
idLabel.Size = UDim2.new(0, 250, 0, 30)
idLabel.Position = UDim2.new(0, 120, 0, 85)
idLabel.BackgroundTransparency = 1
idLabel.TextColor3 = Color3.fromRGB(255,255,255)
idLabel.Font = Enum.Font.Gotham
idLabel.TextSize = 18
idLabel.TextXAlignment = Enum.TextXAlignment.Left
idLabel.Parent = mainFrame

-- Time Label
local timeLabel = Instance.new("TextLabel")
timeLabel.Size = UDim2.new(0, 250, 0, 30)
timeLabel.Position = UDim2.new(0, 10, 0, 170)
timeLabel.BackgroundTransparency = 1
timeLabel.TextColor3 = Color3.fromRGB(255,255,255)
timeLabel.Font = Enum.Font.Gotham
timeLabel.TextSize = 18
timeLabel.TextXAlignment = Enum.TextXAlignment.Left
timeLabel.Parent = mainFrame

-- Players Count
local playersCountLabel = Instance.new("TextLabel")
playersCountLabel.Size = UDim2.new(0, 250, 0, 30)
playersCountLabel.Position = UDim2.new(0, 10, 0, 200)
playersCountLabel.BackgroundTransparency = 1
playersCountLabel.TextColor3 = Color3.fromRGB(255,255,255)
playersCountLabel.Font = Enum.Font.Gotham
playersCountLabel.TextSize = 18
playersCountLabel.TextXAlignment = Enum.TextXAlignment.Left
playersCountLabel.Parent = mainFrame

-- Copy Buttons
local function createCopyButton(position, textToCopy)
    local btn = Instance.new("TextButton")
    btn.Size = UDim2.new(0, 80, 0, 25)
    btn.Position = position
    btn.Text = "نسخ"
    btn.Font = Enum.Font.GothamBold
    btn.TextSize = 16
    btn.BackgroundColor3 = Color3.fromRGB(70,70,70)
    btn.TextColor3 = Color3.fromRGB(255,255,255)
    btn.BorderSizePixel = 0
    btn.Parent = mainFrame
    btn.MouseButton1Click:Connect(function()
        pcall(function() setclipboard(textToCopy) end)
    end)
    return btn
end

local nameBtn = createCopyButton(UDim2.new(0, 340, 0, 50), player.Name)
local idBtn = createCopyButton(UDim2.new(0, 340, 0, 85), tostring(player.UserId))
local timeBtn = createCopyButton(UDim2.new(0, 270, 0, 170), "00 ساعات : 00 دقائق : 00 ثواني")
local playersBtn = createCopyButton(UDim2.new(0, 270, 0, 200), tostring(#Players:GetPlayers()))

-- Set initial info
nameLabel.Text = "الاسم: "..player.Name
idLabel.Text = "معرف اللاعب: "..tostring(player.UserId)
playersCountLabel.Text = "عدد اللاعبين: "..tostring(#Players:GetPlayers())

-- Set thumbnail
local thumbType = Enum.ThumbnailType.AvatarBust
local thumbSize = Enum.ThumbnailSize.Size100x100
local content, isReady = Players:GetUserThumbnailAsync(player.UserId, thumbType, thumbSize)
thumbnail.Image = content

-- Time counter طبيعي
local totalSeconds = 0

local function formatTimeArabic(sec)
    local h = math.floor(sec/3600)
    local m = math.floor((sec%3600)/60)
    local s = sec%60
    return string.format("%02d ساعات : %02d دقائق : %02d ثواني", h, m, s)
end

RunService.RenderStepped:Connect(function(dt)
    totalSeconds = totalSeconds + dt
    local formattedTime = formatTimeArabic(math.floor(totalSeconds))
    timeLabel.Text = "وقت التشغيل: "..formattedTime
    timeBtn.MouseButton1Click:Connect(function()
        pcall(function() setclipboard(formattedTime) end)
    end)
end)

-- Show GUI on chat command
player.Chatted:Connect(function(msg)
    if msg:lower() == ";info" then
        mainFrame.Visible = not mainFrame.Visible
    end
end)

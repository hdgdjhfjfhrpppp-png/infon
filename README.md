--// عداد دخول وخروج لاعبين (حتى 8 اشخاص) + صورة
-- المطور: حسن 👑

local Players = game:GetService("Players")
local ThumbnailType = Enum.ThumbnailType.HeadShot
local ThumbnailSize = Enum.ThumbnailSize.Size100x100

-- GUI
local gui = Instance.new("ScreenGui", game.CoreGui)
gui.Name = "PlayerTrackerGUI"

local frame = Instance.new("Frame", gui)
frame.Size = UDim2.new(0, 420, 0, 500)
frame.Position = UDim2.new(0.3, 0, 0.2, 0)
frame.BackgroundColor3 = Color3.fromRGB(25, 25, 25)
frame.Active, frame.Draggable = true, true

local title = Instance.new("TextLabel", frame)
title.Size = UDim2.new(1, 0, 0, 40)
title.Text = "🔎 عداد دخول وخروج + صورة - المطور حسن 👑"
title.TextColor3 = Color3.new(1,1,1)
title.BackgroundTransparency = 1
title.Font = Enum.Font.SourceSansBold
title.TextScaled = true

-- تخزين
local PlayerStats, Inputs, Labels, Icons = {}, {}, {}, {}
for i = 1, 8 do
    PlayerStats[i] = {Name=nil, Joins=0, Leaves=0}

    local box = Instance.new("TextBox", frame)
    box.Size = UDim2.new(0.35, 0, 0, 30)
    box.Position = UDim2.new(0.05, 0, 0.1 + (i*0.1), 0)
    box.PlaceholderText = "اسم اللاعب "..i
    box.BackgroundColor3 = Color3.fromRGB(40,40,40)
    box.TextColor3 = Color3.new(1,1,1)
    Inputs[i] = box

    local label = Instance.new("TextLabel", frame)
    label.Size = UDim2.new(0.35, 0, 0, 30)
    label.Position = UDim2.new(0.45, 0, 0.1 + (i*0.1), 0)
    label.Text = "دخول: 0 | خروج: 0"
    label.BackgroundTransparency = 1
    label.TextColor3 = Color3.fromRGB(255,255,0)
    Labels[i] = label

    local icon = Instance.new("ImageLabel", frame)
    icon.Size = UDim2.new(0, 30, 0, 30)
    icon.Position = UDim2.new(0.85, 0, 0.1 + (i*0.1), 0)
    icon.BackgroundTransparency = 1
    Icons[i] = icon

    -- كل ما يكتب نص بالبوكس
    box.FocusLost:Connect(function()
        local txt = string.lower(box.Text)
        if txt ~= "" then
            for _, plr in pairs(Players:GetPlayers()) do
                if string.find(string.lower(plr.Name), txt, 1, true) then
                    PlayerStats[i].Name = plr.Name
                    box.Text = plr.Name -- يكمل الاسم تلقائي
                    -- جلب صورة
                    local thumb = Players:GetUserThumbnailAsync(plr.UserId, ThumbnailType, ThumbnailSize)
                    Icons[i].Image = thumb
                    break
                end
            end
        end
    end)
end

-- تحديث العدادات
local function UpdateLabel(i)
    if PlayerStats[i].Name then
        Labels[i].Text = "دخول: "..PlayerStats[i].Joins.." | خروج: "..PlayerStats[i].Leaves
    end
end

Players.PlayerAdded:Connect(function(plr)
    for i = 1,8 do
        if PlayerStats[i].Name == plr.Name then
            PlayerStats[i].Joins += 1
            UpdateLabel(i)
        end
    end
end)

Players.PlayerRemoving:Connect(function(plr)
    for i = 1,8 do
        if PlayerStats[i].Name == plr.Name then
            PlayerStats[i].Leaves += 1
            UpdateLabel(i)
        end
    end
end)

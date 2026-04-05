local Players = game:GetService("Players")
local UIS = game:GetService("UserInputService")
local PlayersService = game:GetService("Players")

-- GUI
local gui = Instance.new("ScreenGui", game.CoreGui)

-- زر العين 👁 كبيرهم
local eyeButton = Instance.new("TextButton", gui)
eyeButton.Size = UDim2.new(0, 130, 0, 40)
eyeButton.Position = UDim2.new(0, 20, 0, 200)
eyeButton.Text = "(👁 كبيرهم)"
eyeButton.TextScaled = true
eyeButton.BackgroundColor3 = Color3.fromRGB(30,30,30)
eyeButton.TextColor3 = Color3.new(1,1,1)

-- الفريم
local frame = Instance.new("Frame", gui)
frame.Size = UDim2.new(0, 450, 0, 450)
frame.Position = UDim2.new(0, 160, 0, 150)
frame.BackgroundColor3 = Color3.fromRGB(15,15,15)
frame.Visible = false
frame.Active = true

-- عنوان
local title = Instance.new("TextLabel", frame)
title.Size = UDim2.new(1, 0, 0, 30)
title.Text = "🔍 تتبع اللاعبين"
title.TextColor3 = Color3.new(1,1,1)
title.BackgroundTransparency = 1
title.TextScaled = true

-- فتح / غلق
eyeButton.MouseButton1Click:Connect(function()
	frame.Visible = not frame.Visible
end)

-- تحريك
local dragging, dragInput, dragStart, startPos
frame.InputBegan:Connect(function(input)
	if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
		dragging = true
		dragStart = input.Position
		startPos = frame.Position
		input.Changed:Connect(function()
			if input.UserInputState == Enum.UserInputState.End then dragging = false end
		end)
	end
end)
frame.InputChanged:Connect(function(input)
	if input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch then
		dragInput = input
	end
end)
UIS.InputChanged:Connect(function(input)
	if input == dragInput and dragging then
		local delta = input.Position - dragStart
		frame.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X,
			startPos.Y.Scale, startPos.Y.Offset + delta.Y)
	end
end)

-- تعريب الاسم
local function arabicName(name)
	local map = {
		a="ا", b="ب", c="ك", d="د", e="ي", f="ف", g="ج",
		h="ه", i="ي", j="ج", k="ك", l="ل", m="م", n="ن",
		o="و", p="ب", q="ق", r="ر", s="س", t="ت",
		u="و", v="ف", w="و", x="كس", y="ي", z="ز"
	}
	local result = ""
	for i=1,#name do
		local ch = name:sub(i,i)
		local low = string.lower(ch)
		result = result .. (map[low] or ch)
	end
	return result
end

-- خانات اللاعبين
local boxes, labels, avatars, data = {}, {}, {}, {}

for i = 1,4 do
	local xPos = (i%2==1) and 0 or 0.5
	local yPos = 40 + (math.floor((i-1)/2)*180)

	-- خانة إدخال الاسم
	local box = Instance.new("TextBox", frame)
	box.Size = UDim2.new(0.45, -10, 0, 30)
	box.Position = UDim2.new(xPos, 10, 0, yPos)
	box.PlaceholderText = "لاعب "..i
	box.BackgroundColor3 = Color3.fromRGB(25,25,25)
	box.TextColor3 = Color3.new(1,1,1)
	table.insert(boxes, box)

	-- صورة اللاعب
	local avatar = Instance.new("ImageLabel", frame)
	avatar.Size = UDim2.new(0.15, 0, 0, 60)
	avatar.Position = UDim2.new(xPos, 10, 0, yPos + 35)
	avatar.BackgroundTransparency = 1
	avatar.Image = ""
	table.insert(avatars, avatar)

	-- نص التتبع
	local label = Instance.new("TextLabel", frame)
	label.Size = UDim2.new(0.3, -10, 0, 60)
	label.Position = UDim2.new(xPos, 70, 0, yPos + 35)
	label.TextColor3 = Color3.new(1,1,1)
	label.BackgroundTransparency = 1
	label.TextWrapped = true
	label.TextXAlignment = Enum.TextXAlignment.Left
	label.TextYAlignment = Enum.TextYAlignment.Top
	label.Text = "..."
	table.insert(labels, label)

	data[i] = {player=nil, startTime=0}

	box.FocusLost:Connect(function()
		local input = box.Text
		for _,p in pairs(Players:GetPlayers()) do
			if string.sub(string.lower(p.Name),1,#input) == string.lower(input) then
				data[i].player = p
				data[i].startTime = tick()
				-- رابط الصورة (Avatar)
				local userId = p.UserId
				avatar.Image = Players:GetUserThumbnailAsync(userId, Enum.ThumbnailType.HeadShot, Enum.ThumbnailSize.Size100x100)
				break
			end
		end
	end)
end

-- تحديث الوقت
task.spawn(function()
	while true do
		for i=1,4 do
			local d = data[i]
			if d.player then
				local exists = Players:FindFirstChild(d.player.Name)
				if exists then
					local t = math.floor(tick() - d.startTime)
					local h = math.floor(t / 3600)
					local m = math.floor((t % 3600) / 60)
					local s = t % 60

					labels[i].Text =
						d.player.Name.." ("..arabicName(d.player.Name)..")\n"
						..h.."س "..m.."د "..s.."ث"
				else
					labels[i].Text = "خرج 1"
					avatars[i].Image = ""
					d.player = nil
				end
			end
		end
		wait(1)
	end
end)

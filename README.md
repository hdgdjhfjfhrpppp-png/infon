-- ===== Orion Full GUI (with your pastebin) =====
local OrionLib = loadstring(game:HttpGet('https://raw.githubusercontent.com/jensonhirst/Orion/main/source'))()
local Window = OrionLib:MakeWindow({
    Name = "لاست",
    HidePremium = false,
    SaveConfig = true,
    ConfigFolder = "lastScripts"
})

-- ===== تبويب سكربتاتي الصملة 🔥 =====
local Tab = Window:MakeTab({
    Name = "سكربتاتي الصملة 🔥",
    Icon = "rbxassetid://4483345998",
    PremiumOnly = false
})

local scriptsList = {
    {"ALSHABA7 VOIID", 'https://raw.githubusercontent.com/XxAbood/ALSHABA7-VOIID/refs/heads/main/ALSHABA7%20VOIID'},
    {"Auto Click", 'https://raw.githubusercontent.com/MADARA9223/AUTO-CLICK/refs/heads/main/MADARA%20AUTO%20CLICK'},
    {"VR7", 'https://raw.githubusercontent.com/VR7ss/OMK/refs/heads/main/VR7-ON-TOP'},
    {"HAMODAH", 'https://raw.githubusercontent.com/SALAH142876/HAMODAH_ON_TOP/refs/heads/main/Protected_7545697692462583.txt'},
    {"AntiAFK", 'https://rawscripts.net/raw/Universal-Script-AntiAFK-v-AntiKick-V3-v-Kick-Attempt-Logger-27977'},
    {"رحمه القمر 🌙", 'https://raw.githubusercontent.com/n0kc/AtomicHub/main/Map-Al-Biout.lua'}
}

for _,v in pairs(scriptsList) do
    Tab:AddButton({
        Name = "تشغيل "..v[1],
        Callback = function()
            pcall(function() loadstring(game:HttpGet(v[2]))() end)
            OrionLib:MakeNotification({Name="تم التشغيل", Content=v[1].." شغال الآن", Image="rbxassetid://4483345998", Time=3})
        end
    })
end

-- === ADDED: your pastebin script button ===
Tab:AddButton({
    Name = "تشغيل (pastebin zk61AmRh)",
    Callback = function()
        pcall(function()
            loadstring(game:HttpGet("https://pastebin.com/raw/zk61AmRh"))()
        end)
        OrionLib:MakeNotification({Name="تم التشغيل", Content="pastebin zk61AmRh شغال", Image="rbxassetid://4483345998", Time=3})
    end
})

-- ===== تبويب لاست — أدوات =====
local LastTab = Window:MakeTab({
    Name = "لاست — أدوات",
    Icon = "rbxassetid://6035027362",
    PremiumOnly = false
})

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local Lighting = game:GetService("Lighting")
local LocalPlayer = Players.LocalPlayer
local Workspace = game:GetService("Workspace")

local state = { antiafk=false, reduceLag=false, antiKickLocal=false, autoMove=false, autokeyActive=false, repLagEnabled=false, reachMultiplier=1, vanished=false }
local savedParts = {}

-- حفظ واسترجاع الشخصية
local function saveCharacterState(char)
    savedParts = {}
    for _,v in pairs(char:GetDescendants()) do
        if v:IsA("BasePart") then savedParts[v]={Transparency=v.Transparency, CanCollide=v.CanCollide}
        elseif v:IsA("Decal") then savedParts[v]={Transparency=v.Transparency} end
    end
end
local function restoreCharacter(char)
    for obj,props in pairs(savedParts) do
        pcall(function()
            if obj and obj.Parent then
                if obj:IsA("BasePart") then obj.Transparency=props.Transparency or 0; obj.CanCollide=(props.CanCollide==nil) and true or props.CanCollide; if rawget(obj,"LocalTransparencyModifier")~=nil then obj.LocalTransparencyModifier=0 end
                elseif obj:IsA("Decal") then obj.Transparency=props.Transparency or 0 end
            end
        end)
    end
    savedParts={}
    state.vanished=false
end

-- وظائف محلية
local antiafkConn
local function enableAntiafk()
    if state.antiafk then return end
    state.antiafk=true
    antiafkConn=RunService.Heartbeat:Connect(function()
        local char=LocalPlayer.Character
        if char and char.PrimaryPart then
            pcall(function() char.PrimaryPart.CFrame=char.PrimaryPart.CFrame+Vector3.new(0,0.03*math.sin(tick()),0) end)
        end
    end)
    OrionLib:MakeNotification({Name="AntiAFK", Content="تم تفعيل AntiAFK", Image="rbxassetid://4483345998", Time=3})
end
local function disableAntiafk() state.antiafk=false; if antiafkConn then antiafkConn:Disconnect(); antiafkConn=nil end; OrionLib:MakeNotification({Name="AntiAFK", Content="تم إيقاف AntiAFK", Image="rbxassetid://4483345998", Time=3}) end

local function enableReduceLag()
    if state.reduceLag then return end
    state.reduceLag=true
    for _,v in pairs(Workspace:GetDescendants()) do
        if v:IsA("ParticleEmitter") or v:IsA("Trail") or v:IsA("Smoke") then pcall(function() v.Enabled=false end) end
    end
    pcall(function() Lighting.GlobalShadows=false end)
    OrionLib:MakeNotification({Name="ReduceLag", Content="تم تقليل العناصر المزعجة", Image="rbxassetid://4483345998", Time=3})
end
local function disableReduceLag() state.reduceLag=false; OrionLib:MakeNotification({Name="ReduceLag", Content="تم إيقاف ReduceLag", Image="rbxassetid://4483345998", Time=3}) end

local function enableAntiKickLocal()
    state.antiKickLocal=true
    Players.LocalPlayer.Idled:Connect(function()
        if state.antiKickLocal and LocalPlayer.Character then
            saveCharacterState(LocalPlayer.Character)
            pcall(function()
                for _,p in pairs(LocalPlayer.Character:GetDescendants()) do
                    if p:IsA("BasePart") then p.Transparency=1; p.CanCollide=false end
                    if p:IsA("Decal") then p.Transparency=1 end
                    if p:IsA("ParticleEmitter") or p:IsA("Trail") then p.Enabled=false end
                end
            end)
            wait(2)
            if LocalPlayer.Character then restoreCharacter(LocalPlayer.Character) end
        end
    end)
    OrionLib:MakeNotification({Name="AntiKickLocal", Content="حماية الطرد مفعلة", Image="rbxassetid://4483345998", Time=3})
end
local function disableAntiKickLocal() state.antiKickLocal=false; OrionLib:MakeNotification({Name="AntiKickLocal", Content="حماية الطرد معطلة", Image="rbxassetid://4483345998", Time=3}) end

local function brightnessNormal() pcall(function() Lighting.Brightness=1 end); OrionLib:MakeNotification({Name="Brightness", Content="Brightness = 1", Image="rbxassetid://4483345998", Time=2}) end
local function brightnessNan() pcall(function() Lighting.Brightness=0.01 end); OrionLib:MakeNotification({Name="Brightness", Content="Brightness = 0.01", Image="rbxassetid://4483345998", Time=2}) end
local function brightnessInf() pcall(function() Lighting.Brightness=12 end); OrionLib:MakeNotification({Name="Brightness", Content="Brightness = 12", Image="rbxassetid://4483345998", Time=2}) end

local guiWhitelist={"Map","HUD","hud","Minimap","MapUI","Brainrot","Menu"}
local function isWhitelisted(g)
    if not g or not g.Name then return false end
    for _,kw in ipairs(guiWhitelist) do if string.find(g.Name,kw) then return true end end
    return false
end
local function nogui()
    for _,g in pairs(LocalPlayer.PlayerGui:GetChildren()) do
        if g:IsA("ScreenGui") and not isWhitelisted(g) then pcall(function() g.Enabled=false end) end
    end
    OrionLib:MakeNotification({Name="NoGUI", Content="تم إخفاء الواجهات", Image="rbxassetid://4483345998", Time=2})
end

local function setfpscap(val) pcall(function() if typeof(settings)=="function" then settings().Physics.ForceFPS=tonumber(val) or 30 end end); OrionLib:MakeNotification({Name="FPS Cap", Content="تم ضبط FPS Cap", Image="rbxassetid://4483345998", Time=2}) end

local repLagThread
local function enableReplicationLag(ms)
    if state.repLagEnabled then return end
    state.repLagEnabled=true
    repLagThread=spawn(function() while state.repLagEnabled do wait((tonumber(ms) or 80)/1000) end end)
    OrionLib:MakeNotification({Name="RepLag", Content="Replication lag مفعل", Image="rbxassetid://4483345998", Time=2})
end
local function disableReplicationLag() state.repLagEnabled=false; repLagThread=nil; OrionLib:MakeNotification({Name="RepLag", Content="Replication lag معطل", Image="rbxassetid://4483345998", Time=2}) end

local function cpuReduce()
    for _,v in pairs(Workspace:GetDescendants()) do
        pcall(function()
            if v:IsA("ParticleEmitter") or v:IsA("Trail") or v:IsA("Smoke") then v.Enabled=false end
            if v:IsA("Decal") then v.Transparency=math.max(v.Transparency or 0,0.5) end
        end)
    end
    pcall(function() Lighting.GlobalShadows=false end)
    OrionLib:MakeNotification({Name="CPU Reduce", Content="تم تقليل حمل الـ CPU", Image="rbxassetid://4483345998", Time=2})
end

local function setDay() pcall(function() Lighting.TimeOfDay="14:00:00" end); OrionLib:MakeNotification({Name="Day", Content="تم ضبط الوقت", Image="rbxassetid://4483345998", Time=2}) end
local function nofog(val) pcall(function() Lighting.FogEnd=tonumber(val) or 0.1 end); OrionLib:MakeNotification({Name="NoFog", Content="تم ضبط NoFog", Image="rbxassetid://4483345998", Time=2}) end
local function antilag()
    pcall(function()
        Lighting.GlobalShadows=false
        Lighting.FogEnd=10000
        for _,v in pairs(Workspace:GetDescendants()) do
            if v:IsA("ParticleEmitter") or v:IsA("Trail") or v:IsA("Smoke") then v.Enabled=false end
            if v:IsA("Decal") then v.Transparency=math.max(v.Transparency or 0,0.5) end
        end
    end)
    OrionLib:MakeNotification({Name="AntiLag", Content="تم تفعيل AntiLag", Image="rbxassetid://4483345998", Time=2})
end

local function setReach(mult)
    local char=LocalPlayer.Character
    if char and char:FindFirstChild("HumanoidRootPart") then
        pcall(function() char.HumanoidRootPart.Size=Vector3.new(2*(mult or 1),2*(mult or 1),1) end)
        state.reachMultiplier=mult or 1
        OrionLib:MakeNotification({Name="Reach", Content="Reach x"..tostring(mult), Image="rbxassetid://4483345998", Time=2})
    end
end

local function autokey_w(times)
    if state.autokeyActive then return end
    state.autokeyActive=true
    spawn(function()
        local t=tonumber(times) or 26
        for i=1,t do
            local char=LocalPlayer.Character
            if char then
                local humanoid=char:FindFirstChildOfClass("Humanoid")
                local hrp=char:FindFirstChild("HumanoidRootPart")
                local cam=workspace.CurrentCamera
                if humanoid and hrp and cam then
                    local dir=cam.CFrame.LookVector
                    local horiz=Vector3.new(dir.X,0,dir.Z)
                    if horiz.Magnitude>0 then
                        local target=hrp.Position+horiz.Unit*4
                        pcall(function() humanoid:MoveTo(Vector3.new(target.X,hrp.Position.Y,target.Z)) end)
                    end
                end
            end
            wait(0.12)
        end
        state.autokeyActive=false
        OrionLib:MakeNotification({Name="Autokey", Content="انتهى Autokey", Image="rbxassetid://4483345998", Time=2})
    end)
end

local autoMoveThread
local function startAutoMove()
    if state.autoMove then return end
    state.autoMove=true
    autoMoveThread=spawn(function()
        while state.autoMove do
            local char=LocalPlayer.Character
            if char and char.PrimaryPart then
                local humanoid=char:FindFirstChildOfClass("Humanoid")
                local hrp=char.PrimaryPart
                local cam=workspace.CurrentCamera
                if humanoid and hrp and cam then
                    local dir=cam.CFrame.LookVector
                    local horiz=Vector3.new(dir.X,0,dir.Z)
                    if horiz.Magnitude>0 then
                        local target=hrp.Position+horiz.Unit*6
                        pcall(function() humanoid:MoveTo(Vector3.new(target.X,hrp.Position.Y,target.Z)) end)
                    end
                end
            end
            wait(0.2)
        end
    end)
    OrionLib:MakeNotification({Name="AutoMove", Content="تحرك تلقائي مفعل", Image="rbxassetid://4483345998", Time=2})
end
local function stopAutoMove() state.autoMove=false; autoMoveThread=nil; OrionLib:MakeNotification({Name="AutoMove", Content="تحرك تلقائي متوقف", Image="rbxassetid://4483345998", Time=2}) end

local function jumpAndVanish(duration)
    duration=tonumber(duration) or 6
    local char=LocalPlayer.Character
    if not char then return end
    local humanoid=char:FindFirstChildOfClass("Humanoid")
    if humanoid then pcall(function() humanoid.Jump=true end) end
    wait(0.12)
    saveCharacterState(char)
    for _,v in pairs(char:GetDescendants()) do
        pcall(function()
            if v:IsA("BasePart") then v.Transparency=1; v.CanCollide=false end
            if v:IsA("Decal") then v.Transparency=1 end
            if v:IsA("ParticleEmitter") or v:IsA("Trail") then v.Enabled=false end
        end)
    end
    state.vanished=true
    OrionLib:MakeNotification({Name="Vanish", Content="تم الاختفاء مؤقتاً", Image="rbxassetid://4483345998", Time=2})
    spawn(function() wait(duration) if LocalPlayer.Character then restoreCharacter(LocalPlayer.Character) end end)
end

-- إضافة أزرار تبويب لاست
local function addButton(name,fn)
    LastTab:AddButton({ Name=name, Callback=function() pcall(fn) end })
end

addButton("AntiAFK (Toggle)", function() if state.antiafk then disableAntiafk() else enableAntiafk() end end)
addButton("تقليل ضغط (Toggle)", function() if state.reduceLag then disableReduceLag() else enableReduceLag() end end)
addButton("حماية طرد (Soft Toggle)", function() if state.antiKickLocal then disableAntiKickLocal() else enableAntiKickLocal() end end)
addButton("brightness عادي", brightnessNormal)
addButton("brightness nan", brightnessNan)
addButton("brightness inf", brightnessInf)
addButton("nogui", nogui)
addButton("FPS 0.1", function() setfpscap(0.1) end)
addButton("replicationlag 80ms", function() if state.repLagEnabled then disableReplicationLag() else enableReplicationLag(80) end end)
addButton("CPU reduce", cpuReduce)
addButton("day", setDay)
addButton("nofog 0.1", function() nofog(0.1) end)
addButton("antilag", antilag)
addButton("reach x2", function() setReach(2) end)
addButton("autokeypress W x26", function() autokey_w(26) end)
addButton("تحرك تلقائي", function() if state.autoMove then stopAutoMove() else startAutoMove() end end)
addButton("قفز + اختفاء", function() jumpAndVanish(6) end)

-- ===== تبويب ضغط / تحسين الأداء =====
local PressureTab = Window:MakeTab({
    Name = "ضغط / تحسين الأداء",
    Icon = "rbxassetid://6035027362",
    PremiumOnly = false
})

PressureTab:AddButton({ Name = "تشغيل سكربت AL7FRAA-MADARAxCATAY", Callback = function()
    pcall(function() loadstring(game:HttpGet("https://raw.githubusercontent.com/MADARA9223/AL7FRAA-MADARAxCATAY/refs/heads/main/AL7FRAA%20%7C%20MADARAxCATAYxROBERTO"))() end)
    OrionLib:MakeNotification({Name="تم التشغيل", Content="AL7FRAA-MADARAxCATAY شغال", Image="rbxassetid://4483345998", Time=3})
end })

-- إشعار جاهزية الواجهة
OrionLib:MakeNotification({
    Name = "جاهز",
    Content = "واجهة لاست تم إنشاؤها",
    Image = "rbxassetid://4483345998",
    Time = 4
})

    local gui = Instance.new("ScreenGui")
    gui.Name = "Last_TrueFly_Watch"
    gui.ResetOnSpawn = false
    gui.Parent = playerGui

    local frame = Instance.new("Frame", gui)
    frame.Size = UDim2.new(0, 380, 0, 300)
    frame.Position = UDim2.new(0.5, -190, 0.06, 0)
    frame.BackgroundColor3 = Color3.fromRGB(18,18,18)
    frame.BorderSizePixel = 0
    frame.Active = true
    frame.Draggable = true

    local title = Instance.new("TextLabel", frame)
    title.Size = UDim2.new(1,0,0,36)
    title.Position = UDim2.new(0,0,0,0)
    title.BackgroundTransparency = 1
    title.Text = "🛸 لاست — مراقبة + بنق + طيران فوك"
    title.Font = Enum.Font.GothamBold
    title.TextSize = 16
    title.TextColor3 = Color3.new(1,1,1)

    local input = Instance.new("TextBox", frame)
    input.Name = "WatchInput"
    input.PlaceholderText = "اكتب أول حرفين من اسم اللاعب..."
    input.Size = UDim2.new(1,-20,0,34)
    input.Position = UDim2.new(0,10,0,45)
    input.BackgroundColor3 = Color3.fromRGB(35,35,35)
    input.TextColor3 = Color3.new(1,1,1)
    input.Font = Enum.Font.Gotham
    input.TextSize = 15

    local status = Instance.new("TextLabel", frame)
    status.Name = "Status"
    status.Size = UDim2.new(1,-20,0,46)
    status.Position = UDim2.new(0,10,0,85)
    status.BackgroundColor3 = Color3.fromRGB(28,28,28)
    status.TextColor3 = Color3.fromRGB(0,220,0)
    status.Font = Enum.Font.Code
    status.TextSize = 14
    status.Text = "جاهز للمراقبة..."

    local pingLabel = Instance.new("TextLabel", frame)
    pingLabel.Name = "PingLabel"
    pingLabel.Size = UDim2.new(1,-20,0,30)
    pingLabel.Position = UDim2.new(0,10,0,140)
    pingLabel.BackgroundColor3 = Color3.fromRGB(18,18,18)
    pingLabel.TextColor3 = Color3.fromRGB(255,210,0)
    pingLabel.Font = Enum.Font.Code
    pingLabel.TextSize = 14
    pingLabel.Text = "Ping: ..."

    local flyBtn = Instance.new("TextButton", frame)
    flyBtn.Name = "FlyBtn"
    flyBtn.Size = UDim2.new(1,-20,0,36)
    flyBtn.Position = UDim2.new(0,10,0,180)
    flyBtn.BackgroundColor3 = Color3.fromRGB(40,40,40)
    flyBtn.Font = Enum.Font.GothamBold
    flyBtn.TextSize = 15
    flyBtn.TextColor3 = Color3.new(1,1,1)
    flyBtn.Text = "🛸 تفعيل الطيران (فوك)"

    local upBtn = Instance.new("TextButton", frame)
    upBtn.Name = "UpBtn"
    upBtn.Size = UDim2.new(0.48,-12,0,32)
    upBtn.Position = UDim2.new(0,10,0,224)
    upBtn.Text = "⬆️ صعود"
    upBtn.Font = Enum.Font.Gotham
    upBtn.BackgroundColor3 = Color3.fromRGB(45,45,45)
    upBtn.TextColor3 = Color3.new(1,1,1)

    local downBtn = Instance.new("TextButton", frame)
    downBtn.Name = "DownBtn"
    downBtn.Size = UDim2.new(0.48,-12,0,32)
    downBtn.Position = UDim2.new(0.52,2,0,224)
    downBtn.Text = "⬇️ نزول"
    downBtn.Font = Enum.Font.Gotham
    downBtn.BackgroundColor3 = Color3.fromRGB(45,45,45)
    downBtn.TextColor3 = Color3.new(1,1,1)

    local credit = Instance.new("TextLabel", frame)
    credit.Size = UDim2.new(1,0,0,22)
    credit.Position = UDim2.new(0,0,1,-22)
    credit.BackgroundTransparency = 1
    credit.Font = Enum.Font.Code
    credit.TextSize = 13
    credit.TextColor3 = Color3.fromRGB(200,200,200)
    credit.Text = "من صنع لاست 🧠"

    -- return table of useful references
    return {
        Gui = gui,
        Frame = frame,
        Input = input,
        Status = status,
        PingLabel = pingLabel,
        FlyBtn = flyBtn,
        UpBtn = upBtn,
        DownBtn = downBtn,
        Credit = credit
    }
end

local flyUI = createFlyGui()

-- function to show gui (Fly tab button)
local FlyTab = Window:MakeTab({
    Name = "🛸 طيران + مراقبة",
    Icon = "rbxassetid://4483345998",
    PremiumOnly = false
})
FlyTab:AddButton({ Name = "فتح واجهة الطيران + مراقبة", Callback = function()
    if flyUI and flyUI.Gui then flyUI.Gui.Parent = playerGui end
end })

-- ====== TrueFly logic ======
local flying = false
local speed = 140          -- default flight horizontal speed
local verticalPower = 80   -- default vertical speed
local control = {F=0,B=0,L=0,R=0,U=0,D=0}
local bv, bg

local function getCharParts()
    local char = LP.Character
    if not char then return nil end
    local hrp = char:FindFirstChild("HumanoidRootPart")
    local hum = char:FindFirstChildOfClass("Humanoid")
    if hrp and hum then return char, hrp, hum end
    return nil
end

local function enableFly()
    local char, hrp, hum = getCharParts()
    if not char then return end
    if bg then bg:Destroy(); bg = nil end
    if bv then bv:Destroy(); bv = nil end

    bg = Instance.new("BodyGyro")
    bg.MaxTorque = Vector3.new(1e8,1e8,1e8)
    bg.P = 1e4
    bg.CFrame = hrp.CFrame
    bg.D = 5
    bg.Parent = hrp

    bv = Instance.new("BodyVelocity")
    bv.MaxForce = Vector3.new(1e8,1e8,1e8)
    bv.Velocity = Vector3.new(0,0,0)
    bv.P = 1e4
    bv.Parent = hrp

    hum.PlatformStand = true
    flying = true
    pcall(function() flyUI.FlyBtn.Text = "🛬 إيقاف الطيران" end)
    pcall(function() flyUI.Status.Text = "🚀 الطيران مفعل" end)
end

local function disableFly()
    flying = false
    pcall(function() flyUI.FlyBtn.Text = "🛸 تفعيل الطيران (فوك)" end)
    pcall(function() flyUI.Status.Text = "🛬 الطيران متوقف" end)
    pcall(function()
        if bg then bg:Destroy(); bg = nil end
        if bv then bv:Destroy(); bv = nil end
        local char, hrp, hum = getCharParts()
        if hum then hum.PlatformStand = false end
    end)
end

-- Button connections (safe pcall to avoid errors)
if flyUI then
    pcall(function()
        flyUI.FlyBtn.MouseButton1Click:Connect(function()
            if flying then disableFly() else enableFly() end
        end)
        flyUI.UpBtn.MouseButton1Click:Connect(function()
            control.U = 1
            wait(0.15)
            control.U = 0
        end)
        flyUI.DownBtn.MouseButton1Click:Connect(function()
            control.D = 1
            wait(0.15)
            control.D = 0
        end)
    end)
end

-- Keyboard controls
UserInputService.InputBegan:Connect(function(input, gpe)
    if gpe then return end
    local k = input.KeyCode
    if k == Enum.KeyCode.W then control.F = 1 end
    if k == Enum.KeyCode.S then control.B = 1 end
    if k == Enum.KeyCode.A then control.L = 1 end
    if k == Enum.KeyCode.D then control.R = 1 end
    if k == Enum.KeyCode.Space then control.U = 1 end
    if k == Enum.KeyCode.LeftControl or k == Enum.KeyCode.LeftShift then control.D = 1 end
end)
UserInputService.InputEnded:Connect(function(input)
    local k = input.KeyCode
    if k == Enum.KeyCode.W then control.F = 0 end
    if k == Enum.KeyCode.S then control.B = 0 end
    if k == Enum.KeyCode.A then control.L = 0 end
    if k == Enum.KeyCode.D then control.R = 0 end
    if k == Enum.KeyCode.Space then control.U = 0 end
    if k == Enum.KeyCode.LeftControl or k == Enum.KeyCode.LeftShift then control.D = 0 end
end)

-- Flight update loop
RunService.RenderStepped:Connect(function(delta)
    if not flying then return end
    local char, hrp, hum = getCharParts()
    if not hrp or not hum then
        disableFly()
        return
    end

    local cam = workspace.CurrentCamera
    if not cam then return end

    local forward = Vector3.new(cam.CFrame.LookVector.X, 0, cam.CFrame.LookVector.Z)
    if forward.Magnitude > 0 then forward = forward.Unit else forward = Vector3.new(0,0,-1) end
    local right = Vector3.new(cam.CFrame.RightVector.X, 0, cam.CFrame.RightVector.Z)
    if right.Magnitude > 0 then right = right.Unit else right = Vector3.new(1,0,0) end

    local mv = Vector3.new(0,0,0)
    mv = mv + forward * (control.F - control.B)
    mv = mv + right * (control.R - control.L)

    local mag = math.clamp(mv.Magnitude, 0, 1)
    local horizontal = (mv.Magnitude > 0) and mv.Unit or Vector3.new(0,0,0)

    local targetVel = Vector3.new(0,0,0)
    if mag > 0 then
        targetVel = horizontal * speed * mag
    end

    local vert = (control.U == 1 and 1 or 0) - (control.D == 1 and 1 or 0)
    targetVel = targetVel + Vector3.new(0, vert * verticalPower, 0)

    if bv and bg then
        pcall(function()
            bv.Velocity = targetVel
            bg.CFrame = CFrame.new(hrp.Position, hrp.Position + Vector3.new(cam.CFrame.LookVector.X, 0, cam.CFrame.LookVector.Z))
        end)
    end
end)

-- Reattach on respawn
LP.CharacterAdded:Connect(function(char)
    wait(0.25)
    if flying then
        disableFly()
        wait(0.05)
        enableFly()
    end
end)

-- Clean on GUI destroy
if flyUI and flyUI.Gui then
    flyUI.Gui.Destroying:Connect(function()
        disableFly()
    end)
end

-- ====== Watch (player monitor) logic ======
local function findPlayerByPrefix(pref)
    if not pref or pref == "" then return nil end
    local lowerPref = string.lower(pref)
    for _,p in pairs(Players:GetPlayers()) do
        if string.sub(string.lower(p.Name), 1, #lowerPref) == lowerPref then
            return p
        end
    end
    return nil
end

if flyUI then
    flyUI.Input.FocusLost:Connect(function(enter)
        if not enter then return end
        local txt = flyUI.Input.Text
        if txt == "" then
            flyUI.Status.Text = "❗ اكتب أول حرفين للبحث"
            return
        end
        local found = findPlayerByPrefix(txt)
        if found then
            flyUI.Status.TextColor3 = Color3.fromRGB(0,255,0)
            flyUI.Status.Text = "🔎 مراقب: " .. found.Name
        else
            flyUI.Status.TextColor3 = Color3.fromRGB(255,200,0)
            flyUI.Status.Text = "⚠️ ما لقيت لاعب"
        end
    end)

    Players.PlayerAdded:Connect(function(p)
        local watch = string.lower(flyUI.Input.Text or "")
        if watch ~= "" and string.sub(string.lower(p.Name),1,#watch) == watch then
            flyUI.Status.TextColor3 = Color3.fromRGB(0,255,0)
            flyUI.Status.Text = "✅ دخل اللاعب: "..p.Name
        end
    end)
    Players.PlayerRemoving:Connect(function(p)
        local watch = string.lower(flyUI.Input.Text or "")
        if watch ~= "" and string.sub(string.lower(p.Name),1,#watch) == watch then
            flyUI.Status.TextColor3 = Color3.fromRGB(255,0,0)
            flyUI.Status.Text = "❌ طلع اللاعب: "..p.Name
        end
    end)
end

-- Ping counter (heartbeat)
RunService.Heartbeat:Connect(function()
    local ok, ping = pcall(function()
        return math.floor(game:GetService("Stats").Network.ServerStatsItem["Data Ping"]:GetValue())
    end)
    if flyUI and flyUI.PingLabel then
        if ok and ping then flyUI.PingLabel.Text = "Ping: " .. tostring(ping) .. " ms"
        else flyUI.PingLabel.Text = "Ping: n/a" end
    end
end)

-- ====== Done ======
-- Final notification
pcall(function()
    OrionLib:MakeNotification({
        Name = "لاست",
        Content = "النسخة الكاملة (Orion + أدوات + طيران) جاهزة",
        Image = "rbxassetid://4483345998",
        Time = 4
    })
end)

-- End of LocalScript

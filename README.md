-- LocalScript filename: لاست
-- واجهة قابلة للسحب + أوامر تعمل محلياً
-- ضع هذا الملف في: StarterPlayer > StarterPlayerScripts

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local Lighting = game:GetService("Lighting")
local LocalPlayer = Players.LocalPlayer

-- ===== إنشاء واجهة بسيطة وقابلة للسحب باللمس =====
local screen = Instance.new("ScreenGui")
screen.Name = "لاستGUI"
screen.ResetOnSpawn = false
screen.Parent = LocalPlayer:WaitForChild("PlayerGui")

local main = Instance.new("Frame")
main.Name = "لاست"
main.Size = UDim2.new(0,330,0,560)
main.Position = UDim2.new(0,20,0,60)
main.BackgroundColor3 = Color3.fromRGB(28,28,28)
main.BorderSizePixel = 0
main.Parent = screen

local title = Instance.new("TextLabel")
title.Parent = main
title.Size = UDim2.new(1,0,0,34)
title.Position = UDim2.new(0,0,0,0)
title.BackgroundTransparency = 1
title.Text = "لاست"
title.TextColor3 = Color3.new(1,1,1)
title.Font = Enum.Font.GothamBold
title.TextSize = 18

local function mkBtn(txt, y)
    local b = Instance.new("TextButton")
    b.Parent = main
    b.Size = UDim2.new(1,-20,0,34)
    b.Position = UDim2.new(0,10,0,y)
    b.BackgroundColor3 = Color3.fromRGB(50,50,50)
    b.TextColor3 = Color3.fromRGB(1,1,1)
    b.Font = Enum.Font.Gotham
    b.TextSize = 14
    b.Text = txt
    return b
end

local status = Instance.new("TextLabel")
status.Parent = main
status.Size = UDim2.new(1,-20,0,20)
status.Position = UDim2.new(0,10,1,-36)
status.BackgroundTransparency = 1
status.Text = "الحالة: جاهز"
status.TextColor3 = Color3.fromRGB(170,170,170)
status.Font = Enum.Font.Gotham
status.TextSize = 13

-- السحب باللمس للواجهة (العنوان مكان السحب)
local dragging, dragInput, dragStart, startPos = false, nil, nil, nil
local function update(input)
    if not dragging or not input.Position then return end
    local delta = input.Position - dragStart
    main.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X,
                              startPos.Y.Scale, startPos.Y.Offset + delta.Y)
end

title.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.Touch or input.UserInputType == Enum.UserInputType.MouseButton1 then
        dragging = true
        dragStart = input.Position
        startPos = main.Position
        input.Changed:Connect(function()
            if input.UserInputState == Enum.UserInputState.End then
                dragging = false
            end
        end)
    end
end)

title.InputChanged:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.Touch or input.UserInputType == Enum.UserInputType.MouseMovement then
        dragInput = input
    end
end)

UserInputService.InputChanged:Connect(function(input)
    if input == dragInput then update(input) end
end)

-- ===== حالات داخلية وإعدادات افتراضية =====
local state = {
    antiafk = false,
    reduceLag = false,
    antiKickLocal = false,
    autoMove = false,
    autokeyActive = false,
    reachMultiplier = 1.0,
}

local savedParts = {}

-- ===== وظائف مساعدة =====
local function safePcall(fn) pcall(fn) end

local function saveCharacterState(char)
    savedParts = {}
    for _,v in pairs(char:GetDescendants()) do
        if v:IsA("BasePart") then
            savedParts[v] = {Transparency = v.Transparency, CanCollide = v.CanCollide}
        elseif v:IsA("Decal") then
            savedParts[v] = {Transparency = v.Transparency}
        end
    end
end

local function restoreCharacter(char)
    for obj,props in pairs(savedParts) do
        pcall(function()
            if obj and obj.Parent then
                if obj:IsA("BasePart") then
                    obj.Transparency = props.Transparency or 0
                    obj.CanCollide = props.CanCollide == nil and true or props.CanCollide
                    if rawget(obj,"LocalTransparencyModifier") ~= nil then obj.LocalTransparencyModifier = 0 end
                elseif obj:IsA("Decal") then
                    obj.Transparency = props.Transparency or 0
                end
            end
        end)
    end
    savedParts = {}
end

-- ===== أوامر فعلية ومميزاتها =====

-- Anti-AFK: حركة صغيرة دورية لتجنب وضع AFK (محلي وآمن)
local antiafkConn
local function enableAntiafk()
    if state.antiafk then return end
    state.antiafk = true
    antiafkConn = RunService.Heartbeat:Connect(function(dt)
        local char = LocalPlayer.Character
        if char and char.PrimaryPart then
            -- لا نغير موضع بشكل كبير، مجرد تعديل صغير في المحور Y الزمني
            pcall(function() char.PrimaryPart.CFrame = char.PrimaryPart.CFrame + Vector3.new(0,0.04 * math.sin(tick()),0) end)
        end
    end)
    status.Text = "الحالة: AntiAFK مفعل"
end
local function disableAntiafk()
    state.antiafk = false
    if antiafkConn then antiafkConn:Disconnect(); antiafkConn = nil end
    status.Text = "الحالة: AntiAFK متوقف"
end

-- Reduce-Lag: تعطيل الجسيمات، التريل، الدخان، وإيقاف الظلال لتقليل الضغط (محلي)
local function enableReduceLag()
    if state.reduceLag then return end
    state.reduceLag = true
    -- تعطيل مؤقت للجزيئات والتريل والدخان
    for _,v in pairs(workspace:GetDescendants()) do
        if v:IsA("ParticleEmitter") or v:IsA("Trail") or v:IsA("Smoke") then
            pcall(function() v.Enabled = false end)
        end
    end
    pcall(function() Lighting.GlobalShadows = false end)
    status.Text = "الحالة: تقليل ضغط مفعل"
end
local function disableReduceLag()
    state.reduceLag = false
    -- لا نعيد التشغيل افتراضياً (حتى لو أردت نستطيع)
    status.Text = "الحالة: تقليل ضغط متوقف"
end

-- Soft "حماية طرد" محلي: عند الشك نختفي محلياً ونوقف بعض التفاعلات المحلية
local function enableAntiKickLocal()
    if state.antiKickLocal then return end
    state.antiKickLocal = true
    -- عند الخمول نعمل vanish محلي بسيط لتقليل ظهورك
    LocalPlayer.Idled:Connect(function()
        if state.antiKickLocal then
            local char = LocalPlayer.Character
            if char then
                saveCharacterState(char)
                pcall(function()
                    for _,part in pairs(char:GetDescendants()) do
                        if part:IsA("BasePart") then
                            part.Transparency = 1
                            part.CanCollide = false
                        elseif part:IsA("Decal") then
                            part.Transparency = 1
                        end
                    end
                end)
                wait(2)
                if char then restoreCharacter(char) end
            end
        end
    end)
    status.Text = "الحالة: حماية طرد (محلي) مفعلة"
end
local function disableAntiKickLocal()
    state.antiKickLocal = false
    status.Text = "الحالة: حماية طرد متوقفة"
end

-- brightness variations (نستخدم قيم آمنة بدلاً من NaN/Inf الحقيقي لتجنب تعطل العرض)
local function setBrightnessNormal() pcall(function() Lighting.Brightness = 1 end); status.Text = "الحالة: سطوع عادي" end
local function setBrightnessLow() pcall(function() Lighting.Brightness = 0.01 end); status.Text = "الحالة: سطوع منخفض (nan محاكاة)" end
local function setBrightnessHigh() pcall(function() Lighting.Brightness = 12 end); status.Text = "الحالة: سطوع عالي (inf محاكاة)" end

-- nogui: إخفاء جميع ScreenGui غير الموجوده في القائمة البيضاء (محلي)
local guiWhitelist = {"Map","HUD","hud","Minimap","MapUI","Brainrot","Menu"}
local function isWhitelisted(g)
    if not g or not g.Name then return false end
    for _,kw in ipairs(guiWhitelist) do
        if string.find(g.Name, kw) then return true end
    end
    return false
end
local function hideAllGui()
    for _,g in pairs(LocalPlayer.PlayerGui:GetChildren()) do
        if g:IsA("ScreenGui") and not isWhitelisted(g) then
            pcall(function() g.Enabled = false end)
        end
    end
    status.Text = "الحالة: واجهات مخفية (nogui)"
end

-- setfpscap: محاولة تعيين ForceFPS إن كانت متاحة (محلي)
local function setFPSCap(val)
    pcall(function()
        if typeof(settings) == "function" then
            settings().Physics.ForceFPS = tonumber(val) or 30
            status.Text = "الحالة: setfpscap → "..tostring(val)
        else
            status.Text = "الحالة: setfpscap غير مدعوم"
        end
    end)
end

-- replicationlag: عملية محلية تؤخّر حلقة داخلية (لا تغيّر الشبكة الحقيقية للسيرفر)
local repLagEnabled = false
local repLagThread
local function enableReplicationLag(ms)
    if repLagEnabled then return end
    repLagEnabled = true
    repLagThread = spawn(function()
        while repLagEnabled do
            wait((tonumber(ms) or 80)/1000)
            -- مجرد انتظار لتقليل تكرار عمليات محلية إن لزم
        end
    end)
    status.Text = "الحالة: replicationlag "..tostring(ms).."ms مفعل"
end
local function disableReplicationLag()
    repLagEnabled = false
    repLagThread = nil
    status.Text = "الحالة: replicationlag متوقف"
end

-- cpu: محاولة تقليل استهلاك المعالج عبر تعطيل تأثيرات محلية
local function reduceCPU()
    for _,v in pairs(workspace:GetDescendants()) do
        pcall(function()
            if v:IsA("ParticleEmitter") or v:IsA("Trail") or v:IsA("Smoke") then v.Enabled = false end
            if v:IsA("Texture") or v:IsA("Decal") then
                if v.Parent and v.Parent:IsA("BasePart") then
                    -- خفض جودة الصور عبر رفع الشفافية قليلاً (محلي)
                    if v:IsA("Decal") then v.Transparency = math.min((v.Transparency or 0) + 0.5,1) end
                end
            end
        end)
    end
    pcall(function() Lighting.GlobalShadows = false end)
    status.Text = "الحالة: cpu منخفض"
end

-- day: تغيير الوقت إلى نهار محلياً
local function setDay()
    pcall(function() Lighting.TimeOfDay = "14:00:00" end)
    status.Text = "الحالة: نهار"
end

-- nofog: ضبط FogEnd صغير لتحسين الرؤية
local function setNoFog(val)
    pcall(function() Lighting.FogEnd = tonumber(val) or 0.1 end)
    status.Text = "الحالة: nofog "..tostring(val)
end

-- antilag: مجموعة إعدادات لتقليل التأخير محلياً
local function antiLagAll()
    pcall(function()
        Lighting.GlobalShadows = false
        Lighting.FogEnd = 10000
        for _,v in pairs(workspace:GetDescendants()) do
            if v:IsA("ParticleEmitter") or v:IsA("Trail") or v:IsA("Smoke") then
                v.Enabled = false
            end
            if v:IsA("Decal") then
                v.Transparency = math.max(v.Transparency or 0, 0.5)
            end
        end
    end)
    status.Text = "الحالة: antilag مفعل"
end

-- reach: زيادة مدى محلي (نغير حجم HumanoidRootPart بشكل خفيف محلياً)
local function setReach(mult)
    local char = LocalPlayer.Character
    if char and char:FindFirstChild("HumanoidRootPart") then
        pcall(function()
            local hrp = char.HumanoidRootPart
            hrp.Size = Vector3.new(2 * (mult or 1), 2 * (mult or 1), 1)
        end)
        state.reachMultiplier = mult or 1.0
        status.Text = "الحالة: reach x"..tostring(state.reachMultiplier)
    end
end

-- autokeypress w 26: نمشي للأمام (محلي) 26 مرة
local autokeyThread
local function autokeypress_W(times)
    if state.autokeyActive then return end
    state.autokeyActive = true
    autokeyThread = spawn(function()
        local t = tonumber(times) or 26
        for i=1,t do
            local char = LocalPlayer.Character
            if char then
                local humanoid = char:FindFirstChildOfClass("Humanoid")
                local hrp = char:FindFirstChild("HumanoidRootPart")
                if humanoid and hrp then
                    local cam = workspace.CurrentCamera
                    if cam then
                        local dir = cam.CFrame.LookVector
                        local horiz = Vector3.new(dir.X,0,dir.Z)
                        if horiz.Magnitude > 0 then
                            local target = hrp.Position + horiz.Unit * 4
                            pcall(function() humanoid:MoveTo(Vector3.new(target.X, hrp.Position.Y, target.Z)) end)
                        end
                    end
                end
            end
            wait(0.12)
        end
        state.autokeyActive = false
        status.Text = "الحالة: autokeypress منتهي"
    end)
    status.Text = "الحالة: autokeypress يعمل"
end

-- AutoMove: حركة مستمرة للأمام بالنسبة للكاميرا (آمنة، يمكن إيقافها)
local autoMoveThread
local function startAutoMove()
    if state.autoMove then return end
    state.autoMove = true
    autoMoveThread = spawn(function()
        while state.autoMove do
            local char = LocalPlayer.Character
            if char and char.PrimaryPart then
                local humanoid = char:FindFirstChildOfClass("Humanoid")
                local hrp = char.PrimaryPart
                local cam = workspace.CurrentCamera
                if humanoid and hrp and cam then
                    local dir = cam.CFrame.LookVector
                    local horiz = Vector3.new(dir.X,0,dir.Z)
                    if horiz.Magnitude > 0 then
                        local target = hrp.Position + horiz.Unit * 6
                        pcall(function() humanoid:MoveTo(Vector3.new(target.X, hrp.Position.Y, target.Z)) end)
                    end
                end
            end
            wait(0.20)
        end
    end)
    status.Text = "الحالة: تحرك تلقائي مفعل"
end
local function stopAutoMove()
    state.autoMove = false
    autoMoveThread = nil
    status.Text = "الحالة: تحرك تلقائي متوقف"
end

-- Jump + Vanish (محلي): قفزة ثم إخفاء مظهر محلياً مع استعادة تلقائية قصيرة
local function jumpAndVanish(duration)
    duration = tonumber(duration) or 6
    local char = LocalPlayer.Character
    if not char then return end
    local humanoid = char:FindFirstChildOfClass("Humanoid")
    if humanoid then pcall(function() humanoid.Jump = true end) end
    wait(0.12)
    saveCharacterState(char)
    do
        for _,v in pairs(char:GetDescendants()) do
            pcall(function()
                if v:IsA("BasePart") then v.Transparency = 1; v.CanCollide = false end
                if v:IsA("Decal") then v.Transparency = 1 end
                if v:IsA("ParticleEmitter") or v:IsA("Trail") then v.Enabled = false end
            end)
        end
    end
    state.vanished = true
    status.Text = "الحالة: مختفي محلياً"
    spawn(function()
        wait(duration)
        if LocalPlayer.Character then
            restoreCharacter(LocalPlayer.Character)
            state.vanished = false
            status.Text = "الحالة: ظهر"
        end
    end)
end

-- ===== ربط الأزرار بالوظائف =====
local btns = {}

btns["Antiafk"] = mkBtn("Antiafk", 48)
btns["تقليل ضغط"] = mkBtn("تقليل ضغط", 96)
btns["حماية طرد"] = mkBtn("حماية طرد", 144)
btns["brightness"] = mkBtn("brightness (عادي)", 192)
btns["brightness nan"] = mkBtn("brightness nan (محاكاة)", 240)
btns["brightness inf"] = mkBtn("brightness inf (محاكاة)", 288)
btns["nogui"] = mkBtn("nogui (إخفاء واجهات)", 336)
btns["setfpscap(0.1)"] = mkBtn("setfpscap → 0.1", 384)
btns["replicationlag 80"] = mkBtn("replicationlag → 80ms", 432)
btns["cpu"] = mkBtn("cpu (خفض)", 480)
-- إذا احتجت أزرار إضافية أنشئهم هنا (وضعنا مساحة 560 ارتفاع، يمكن التعديل)

-- وظائف أزرار إضافية تحت العلبة (reach, antilag, day, nofog, antilag, reach, autokeypress)
local extraY = 520
local btnReach = mkBtn("reach x2", extraY); extraY = extraY + 44
local btnAntiLag = mkBtn("antilag", extraY); extraY = extraY + 44
local btnDay = mkBtn("day", extraY); extraY = extraY + 44
local btnNoFog = mkBtn("nofog 0.1", extraY); extraY = extraY + 44
local btnAutoKey = mkBtn("autokeypress w ×26", extraY); extraY = extraY + 44
local btnAutoMove = mkBtn("تحرك تلقائي (تبديل)", extraY); extraY = extraY + 44
local btnJumpVanish = mkBtn("قفز + اختفاء", extraY); extraY = extraY + 44

-- Hookups
btns["Antiafk"].MouseButton1Click:Connect(function()
    if state.antiafk then disableAntiafk() else enableAntiafk() end
end)

btns["تقليل ضغط"].MouseButton1Click:Connect(function()
    if state.reduceLag then disableReduceLag() else enableReduceLag() end
end)

btns["حماية طرد"].MouseButton1Click:Connect(function()
    if state.antiKickLocal then disableAntiKickLocal() else enableAntiKickLocal() end
end)

btns["brightness"].MouseButton1Click:Connect(setBrightnessNormal)
btns["brightness nan"].MouseButton1Click:Connect(setBrightnessLow)
btns["brightness inf"].MouseButton1Click:Connect(setBrightnessHigh)
btns["nogui"].MouseButton1Click:Connect(hideAllGui)
btns["setfpscap(0.1)"].MouseButton1Click:Connect(function() setFPSCap(0.1) end)
btns["replicationlag 80"].MouseButton1Click:Connect(function()
    if repLagEnabled then disableReplicationLag() else enableReplicationLag(80) end
end)
btns["cpu"].MouseButton1Click:Connect(reduceCPU)

btnReach.MouseButton1Click:Connect(function() setReach(2) end)
btnAntiLag.MouseButton1Click:Connect(antiLagAll)
btnDay.MouseButton1Click:Connect(setDay)
btnNoFog.MouseButton1Click:Connect(function() setNoFog(0.1) end)
btnAutoKey.MouseButton1Click:Connect(function() autokeypress_W(26) end)
btnAutoMove.MouseButton1Click:Connect(function()
    if state.autoMove then stopAutoMove() else startAutoMove() end
end)
btnJumpVanish.MouseButton1Click:Connect(function() jumpAndVanish(6) end)

-- اختصارات لوحة مفاتيح (M لتبديل AutoMove، J للقفز+اختفاء)
UserInputService.InputBegan:Connect(function(input, gp)
    if gp then return end
    if input.KeyCode == Enum.KeyCode.M then
        if state.autoMove then stopAutoMove() else startAutoMove() end
    elseif input.KeyCode == Enum.KeyCode.J then
        jumpAndVanish(6)
    end
end)

-- إعادة تطبيق بعض الحالات بعد تجدد الشخصية
LocalPlayer.CharacterAdded:Connect(function(char)
    wait(0.5)
    if state.vanished then
        spawn(function() 
            saveCharacterState(char)
            for _,v in pairs(char:GetDescendants()) do
                pcall(function()
                    if v:IsA("BasePart") then v.Transparency=1; v.CanCollide=false end
                    if v:IsA("Decal") then v.Transparency=1 end
                end)
            end
        end)
    end
    if state.reduceLag then enableReduceLag() end
    if state.reachMultiplier and state.reachMultiplier > 1 then setReach(state.reachMultiplier) end
end)

-- نهاية: واجهة جاهزة وبلا مراجع للمصدر
status.Text = "الحالة: جاهز"

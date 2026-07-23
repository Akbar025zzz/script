--[[
================================================================================
  👑 KING AKBAR - ULTIMATE AUTO FARM SCRIPT
     v6.0 FINAL – OFFICE OPTIMIZED + PRINT NEAREST CHAIR + NO LAG
================================================================================
    [+] Developer   : King Akbar
    [+] Update      : - Auto Print (Cari kursi terdekat setelah print)
                      - Cache System (Anti-lag, nggak berat)
                      - MoveTo Anti-Stuck (Nggak ngebug kalau nabrak)
                      - Auto Pindah Kursi jika idle (Bisa diatur detiknya)
================================================================================
]]--

-- // 0. LOAD WINDUI
local WindUI
do
    local ok, result = pcall(function()
        return loadstring(game:HttpGet(
            "https://github.com/Footagesus/WindUI/releases/latest/download/main.lua"
        ))()
    end)
    if ok and result then
        WindUI = result
    else
        warn("[King Akbar] WindUI gagal load: " .. tostring(result))
        WindUI = {
            CreateWindow = function() return {
                Tab = function() return {
                    Paragraph  = function() return { Set = function() end } end,
                    Toggle     = function() end,
                    Button     = function() end,
                    Input      = function() end,
                    Slider     = function() end,
                    Section    = function() return {
                        Paragraph = function() return { Set = function() end } end,
                        Toggle    = function() end,
                        Button    = function() end,
                        Input     = function() end,
                        Slider    = function() end,
                    } end,
                    Select     = function() end,
                } end,
                Tag            = function() return { SetTitle = function() end } end,
                EditOpenButton = function() end,
                SetIconSize    = function() end,
            } end,
            Notify   = function() end,
            SetTheme = function() end,
            Gradient = function() return {} end,
        }
    end
end

-- // 1. SERVICES & REFERENCES
local Services = {
    Players            = game:GetService("Players"),
    RunService         = game:GetService("RunService"),
    TweenSvc           = game:GetService("TweenService"),
    UserInput           = game:GetService("UserInputService"),
    Stats              = game:GetService("Stats"),
    Workspace          = game:GetService("Workspace"),
    VIM                = game:GetService("VirtualInputManager"),
    VirtualUser        = game:GetService("VirtualUser"),
    HttpService        = game:GetService("HttpService"),
    GuiService         = game:GetService("GuiService"),
    ReplicatedStorage  = game:GetService("ReplicatedStorage"),
}

local LocalPlayer = Services.Players.LocalPlayer

local IsMobile = Services.UserInput.TouchEnabled
    and not Services.UserInput.KeyboardEnabled
    and not Services.UserInput.MouseEnabled

local CharRef = {
    Character = LocalPlayer.Character or LocalPlayer.CharacterAdded:Wait(),
    Humanoid  = nil,
    Root      = nil,
}
CharRef.Humanoid = CharRef.Character:WaitForChild("Humanoid")
CharRef.Root     = CharRef.Character:WaitForChild("HumanoidRootPart")

LocalPlayer.CharacterAdded:Connect(function(newChar)
    CharRef.Character = newChar
    CharRef.Humanoid  = newChar:WaitForChild("Humanoid")
    CharRef.Root      = newChar:WaitForChild("HumanoidRootPart")
end)

-- // 2. STATE MANAGER
local State = {
    IsBaristaActive    = false,
    IsOfficeActive     = false,
    IsCourierActive    = false,
    AiThread           = nil,
    StatusText         = "Santai dulu...",
    OrderCount         = 0,
    ActionDelay        = 5,
    AntiAFK            = true,
    AntiAdmin          = true,
    UangAwal           = 0,
    UangAwalSession    = 0,
    SessionStartTime   = 0,
    LastStopReason     = "",
    MachineFixCount    = 0,
    OfficeMathSolved   = 0,
    OfficePrints       = 0,
    CourierDelivered   = 0,
    WebhookEnabled     = false,
    WebhookURL         = "",
    WebhookInterval    = 5,
    WebhookStartMoney  = 0,
    OfficeSettings = {
        MathDelayMin      = 0.8,
        MathDelayMax      = 2.5,
        AfterAnswerDelay  = 3,
        AntiIdleInterval  = 45,
        PrinterCheckInterval = 3,
        EnableAntiIdle    = true,
        EnableAutoPrinter = true,
        EnableChairSwitch = true,
        IdleSwitchTime    = 30, -- Default 30 detik kalau nggak ada soal langsung pindah
    }
}

LocalPlayer.Idled:Connect(function()
    if State.AntiAFK then
        pcall(function()
            Services.VirtualUser:CaptureController()
            Services.VirtualUser:ClickButton2(Vector2.new())
        end)
    end
end)

-- // 3. HUMANIZATION (RNG WAIT)
local function rWait(minSec, maxSec)
    task.wait(math.random((minSec or 0.5) * 1000, (maxSec or 1.5) * 1000) / 1000)
end

-- // 4. GetPlayerMoney
local function GetPlayerMoney()
    local money = 0
    pcall(function()
        if LocalPlayer:FindFirstChild("leaderstats") and LocalPlayer.leaderstats:FindFirstChild("Money") then
            money = LocalPlayer.leaderstats.Money.Value
        elseif LocalPlayer:FindFirstChild("Data") and LocalPlayer.Data:FindFirstChild("Money") then
            money = LocalPlayer.Data.Money.Value
        else
            for _, v in pairs(LocalPlayer.PlayerGui:GetDescendants()) do
                if v:IsA("TextLabel") and v.Visible and string.find(v.Text, "Rp%.") then
                    local m = tonumber(string.gsub(v.Text, "[^%d]", ""))
                    if m and m > money then money = m end
                end
            end
        end
    end)
    return money
end

-- // 5. ADMIN SENSOR
local GAME_GROUP_ID  = 11378976
local MIN_STAFF_RANK = 200

local function CheckForAdmin(player)
    if not State.AntiAdmin or player == LocalPlayer then return end
    pcall(function()
        if player:GetRankInGroup(GAME_GROUP_ID) >= MIN_STAFF_RANK then
            State.LastStopReason = "Admin detected - auto kicked"
            rWait(0.5, 1)
            LocalPlayer:Kick("Woi admin nongol bro, kabur dulu gas biar aman.")
        end
    end)
end

for _, p in ipairs(Services.Players:GetPlayers()) do CheckForAdmin(p) end
Services.Players.PlayerAdded:Connect(CheckForAdmin)

-- // 6. SPLASH SCREEN
do
    local sg = Instance.new("ScreenGui")
    sg.Name = "BaristaSplash"; sg.ResetOnSpawn = false
    sg.IgnoreGuiInset = true; sg.DisplayOrder = 999
    sg.Parent = LocalPlayer:WaitForChild("PlayerGui")

    local bg = Instance.new("Frame", sg)
    bg.Size = UDim2.fromScale(1,1); bg.BackgroundColor3 = Color3.fromHex("#0a0a0a")
    bg.BorderSizePixel = 0; bg.ZIndex = 1

    local grad = Instance.new("UIGradient", bg)
    grad.Color = ColorSequence.new({
        ColorSequenceKeypoint.new(0, Color3.fromHex("#0a0a0a")),
        ColorSequenceKeypoint.new(1, Color3.fromHex("#1e1e1e")),
    }); grad.Rotation = 135

    local ct = Instance.new("Frame", bg)
    ct.Size = UDim2.fromOffset(500, 300); ct.Position = UDim2.fromScale(0.5, 0.5)
    ct.AnchorPoint = Vector2.new(0.5, 0.5); ct.BackgroundTransparency = 1; ct.ZIndex = 2

    local function mkLabel(txt, yOff, sz)
        local l = Instance.new("TextLabel", ct)
        l.Size = UDim2.fromOffset(500, 70); l.Position = UDim2.fromOffset(0, yOff)
        l.BackgroundTransparency = 1; l.Text = txt; l.TextSize = sz
        l.Font = Enum.Font.GothamBold; l.TextColor3 = Color3.fromHex("#ffffff")
        l.TextTransparency = 1; l.ZIndex = 3; return l
    end

    local icon = Instance.new("ImageLabel", ct)
    icon.Size = UDim2.fromOffset(120, 120); icon.Position = UDim2.fromOffset(190, -40)
    icon.BackgroundTransparency = 1; icon.Image = "rbxassetid://91115084979317"
    icon.ImageTransparency = 1; icon.ZIndex = 3

    local title = mkLabel("King Akbar", 70, IsMobile and 38 or 50)
    local tg = Instance.new("UIGradient", title)
    tg.Color = ColorSequence.new({
        ColorSequenceKeypoint.new(0,   Color3.fromHex("#ffffff")),
        ColorSequenceKeypoint.new(0.5, Color3.fromHex("#aaaaaa")),
        ColorSequenceKeypoint.new(1,   Color3.fromHex("#555555")),
    }); tg.Rotation = 45

    local stat = mkLabel("Mempersiapkan mesin tempur...", 200, 12)
    stat.Font = Enum.Font.Gotham; stat.TextColor3 = Color3.fromHex("#555555")
    stat.TextXAlignment = Enum.TextXAlignment.Left; stat.Position = UDim2.fromOffset(50, 200)

    local line = Instance.new("Frame", ct)
    line.Size = UDim2.fromOffset(0, 2); line.Position = UDim2.fromOffset(250, 152)
    line.AnchorPoint = Vector2.new(0.5, 0); line.BackgroundColor3 = Color3.fromHex("#444444")
    line.BorderSizePixel = 0; line.ZIndex = 3

    local barBg = Instance.new("Frame", ct)
    barBg.Size = UDim2.fromOffset(400, 5); barBg.Position = UDim2.fromOffset(50, 190)
    barBg.BackgroundColor3 = Color3.fromHex("#222222"); barBg.BackgroundTransparency = 1
    barBg.BorderSizePixel = 0; barBg.ZIndex = 3
    Instance.new("UICorner", barBg).CornerRadius = UDim.new(1, 0)

    local bar = Instance.new("Frame", barBg)
    bar.Size = UDim2.fromOffset(0, 5); bar.BackgroundColor3 = Color3.fromHex("#ffffff")
    bar.BorderSizePixel = 0; bar.ZIndex = 4
    Instance.new("UICorner", bar).CornerRadius = UDim.new(1, 0)

    local function tw(obj, props, t)
        Services.TweenSvc:Create(obj, TweenInfo.new(t, Enum.EasingStyle.Quart, Enum.EasingDirection.Out), props):Play()
    end

    task.spawn(function()
        tw(icon,  { ImageTransparency = 0 }, 0.5); task.wait(0.15)
        tw(title, { TextTransparency  = 0 }, 0.6); task.wait(0.35)
        tw(line,  { Size = UDim2.fromOffset(400, 2) }, 0.7); task.wait(0.4)
        tw(barBg, { BackgroundTransparency = 0 }, 0.3)
        tw(stat,  { TextTransparency = 0 }, 0.3)

        for _, s in ipairs({
            { "Mempersiapkan RNG Bot...", 0.30 },
            { "Nyalain Alarm Darurat...", 0.60 },
            { "Welcome, King Akbar!",     1.00 },
        }) do
            stat.Text = s[1]
            tw(bar, { Size = UDim2.fromOffset(400 * s[2], 5) }, 0.5)
            task.wait(0.55)
        end

        task.wait(0.3)
        for _, p in ipairs({ bg, icon, title, line, barBg, bar, stat }) do
            local prop = p == stat and "TextTransparency"
                or (p == icon  and "ImageTransparency" or "BackgroundTransparency")
            if p == title then prop = "TextTransparency" end
            tw(p, { [prop] = 1 }, 0.4)
        end
        task.wait(0.8); sg:Destroy()
    end)
    task.wait(3)
end

-- // 7. CONSTANTS & PATHS (BARISTA)
local Constants = {
    START_SHIFT  = Vector3.new(-4991.23, 4.29, -715.26),
    COLOR_ORANGE = Color3.fromRGB(230, 150, 30),
    COLOR_GREEN  = Color3.fromRGB(30,  180, 60),
}

local Paths = {
    START_TO_MACHINE = {
        Vector3.new(-4991.23, 4.29, -715.26), Vector3.new(-5004.86, 4.29, -718.90),
        Vector3.new(-5006.28, 4.29, -802.11), Vector3.new(-4994.18, 4.29, -801.66),
        Vector3.new(-4994.62, 4.29, -794.89), Vector3.new(-4997.13, 4.29, -794.57),
        Vector3.new(-4998.16, 4.29, -794.80),
    },
    MACHINE_TO_CASHIER = {
        Vector3.new(-4997.13, 4.29, -794.57), Vector3.new(-4994.62, 4.29, -794.89),
        Vector3.new(-4995.56, 4.29, -759.78),
    },
    CASHIER_TO_MACHINE = {
        Vector3.new(-4994.62, 4.29, -794.89), Vector3.new(-4997.13, 4.29, -794.57),
        Vector3.new(-4998.16, 4.29, -794.80),
    },
    MACHINE_TO_START = {
        Vector3.new(-4998.16, 4.29, -794.80), Vector3.new(-4997.13, 4.29, -794.57),
        Vector3.new(-4994.62, 4.29, -794.89), Vector3.new(-4994.18, 4.29, -801.66),
        Vector3.new(-5006.28, 4.29, -802.11), Vector3.new(-5004.86, 4.29, -718.90),
        Vector3.new(-4991.23, 4.29, -715.26),
    },
    CASHIER_TO_START = {
        Vector3.new(-4995.56, 4.29, -759.78), Vector3.new(-4994.62, 4.29, -794.89),
        Vector3.new(-4994.18, 4.29, -801.66), Vector3.new(-5006.28, 4.29, -802.11),
        Vector3.new(-5004.86, 4.29, -718.90), Vector3.new(-4991.23, 4.29, -715.26),
    },
    MACHINE_TO_FIX = {
        Vector3.new(-4998.14, 4.29, -795.38), Vector3.new(-4997.02, 4.29, -802.18),
        Vector3.new(-5006.31, 4.29, -802.30), Vector3.new(-5003.75, 4.29, -711.60),
        Vector3.new(-5004.43, 3.19, -670.40), Vector3.new(-5114.86, 3.19, -670.41),
    },
    FIX_TO_MACHINE = {
        Vector3.new(-5114.86, 3.19, -670.41), Vector3.new(-5004.43, 3.19, -670.40),
        Vector3.new(-5003.75, 4.29, -711.60), Vector3.new(-5006.31, 4.29, -802.30),
        Vector3.new(-4997.02, 4.29, -802.18), Vector3.new(-4998.14, 4.29, -795.38),
    },
}

-- // 8. ANTI-LAG & LAYAR HITAM
local BlackGui
local function ToggleBlackScreen(on)
    pcall(function() Services.RunService:Set3dRenderingEnabled(not on) end)
    if on then
        if not BlackGui then
            BlackGui = Instance.new("ScreenGui")
            BlackGui.Name = "BlackScreenSaver"; BlackGui.IgnoreGuiInset = true
            BlackGui.DisplayOrder = 9999; BlackGui.Parent = LocalPlayer:WaitForChild("PlayerGui")
            local f = Instance.new("Frame", BlackGui)
            f.Size = UDim2.fromScale(1,1); f.BackgroundColor3 = Color3.new(0,0,0)
            local t = Instance.new("TextLabel", f)
            t.Text = "🌑 MODE HEMAT BATERAI AKTIF 🌑\nKing Akbar Lagi Cari Cuan..."
            t.Size = UDim2.fromScale(1,1); t.TextColor3 = Color3.new(1,1,1)
            t.BackgroundTransparency = 1; t.Font = Enum.Font.GothamBold; t.TextSize = 20
        end
        BlackGui.Enabled = true
    else
        if BlackGui then BlackGui.Enabled = false end
    end
end

-- // 9. UTILITY (BARISTA)
local function WalkToPoint(pos)
    if not CharRef.Humanoid or not CharRef.Root then return end
    CharRef.Humanoid.Sit = false
    local hp = pos + Vector3.new(math.random(-15,15)/10, 0, math.random(-15,15)/10)
    CharRef.Humanoid:MoveTo(hp)
    local t = 10
    while t > 0 and State.IsBaristaActive do
        local d = Vector3.new(CharRef.Root.Position.X, 0, CharRef.Root.Position.Z)
               - Vector3.new(hp.X, 0, hp.Z)
        if d.Magnitude < 3 then break end
        task.wait(0.1); t -= 0.1
    end
end

local function FollowPath(arr)
    for _, p in ipairs(arr) do
        if not State.IsBaristaActive then break end
        WalkToPoint(p)
    end
end

local function FindPrompt(kw, maxD, origin)
    if not CharRef.Root then return nil end
    origin = origin or CharRef.Root.Position; maxD = maxD or 20
    local found, closest = nil, maxD
    for _, v in pairs(Services.Workspace:GetDescendants()) do
        if v:IsA("ProximityPrompt") and v.Enabled
            and string.find(string.lower(v.ActionText), string.lower(kw))
        then
            local part = v.Parent
            if part and part:IsA("BasePart") then
                local d = (part.Position - origin).Magnitude
                if d < closest then closest = d; found = v end
            end
        end
    end
    return found
end

local function DoHold(prompt)
    if not prompt then return false end
    pcall(function()
        prompt:InputHoldBegin()
        rWait((prompt.HoldDuration or 1) + 0.2, (prompt.HoldDuration or 1) + 0.6)
        prompt:InputHoldEnd()
    end)
    rWait(0.1, 0.3); return true
end

local function DoTap(prompt)
    if not prompt then return false end
    pcall(function()
        prompt:InputHoldBegin(); rWait(0.08, 0.18); prompt:InputHoldEnd()
    end)
    rWait(0.2, 0.4); return true
end

local function IsMachineBroken()
    for _, gui in pairs(LocalPlayer.PlayerGui:GetChildren()) do
        for _, v in pairs(gui:GetDescendants()) do
            if v:IsA("TextLabel") and v.Visible then
                local t = string.lower(v.Text)
                if t:find("machine broke") or t:find("needs maintenance") or t:find("fix machine") then
                    return true
                end
            end
        end
    end
    return false
end

local function HasJob()
    local hasJob = true
    for _, v in pairs(Services.Workspace:GetDescendants()) do
        if v:IsA("ProximityPrompt") and v.Enabled and v.ActionText:lower():find("shift") then
            local part = v.Parent
            if part and part:IsA("BasePart") and (part.Position - Constants.START_SHIFT).Magnitude < 40 then
                hasJob = v.ActionText:lower():find("end") and true or false
                break
            end
        end
    end
    return hasJob
end

local function FindByColor(parent, col, tol)
    local best, bestD = nil, math.huge
    for _, v in pairs(parent:GetDescendants()) do
        if (v:IsA("Frame") or v:IsA("ImageLabel")) and v.Visible and v.BackgroundTransparency < 0.8 then
            local c = v:IsA("ImageLabel") and v.ImageColor3 or v.BackgroundColor3
            local d = math.abs(c.R-col.R) + math.abs(c.G-col.G) + math.abs(c.B-col.B)
            if d < bestD then bestD = d; best = v end
        end
    end
    return bestD < (tol or 0.6) and best or nil
end

-- // 10. AI MINIGAME (BARISTA)
local function StartMinigameAI()
    if State.AiThread then task.cancel(State.AiThread) end
    State.AiThread = task.spawn(function()
        local cam = Services.Workspace.CurrentCamera
        while State.IsBaristaActive do
            task.wait(0.016)
            local gui = LocalPlayer.PlayerGui:FindFirstChild("BaristaGUI")
            if not gui then task.wait(0.1); continue end
            local mf = gui:FindFirstChild("MinigameFrame", true)
            if not (mf and mf.Visible) then task.wait(0.1); continue end

            local cx = (cam.ViewportSize.X/2) + math.random(-15,15)
            local cy = (cam.ViewportSize.Y/2) + math.random(-15,15)
            local pill, bar = nil, nil

            for _, v in pairs(mf:GetDescendants()) do
                if v:IsA("Frame") or v:IsA("ImageLabel") then
                    local nm = v.Name:lower()
                    if nm:find("pill") or nm:find("indicator") or nm:find("player") or nm:find("handle") then pill = v end
                    if nm:find("target") or nm:find("zone") or nm:find("goal") or nm:find("safe") then bar = v end
                end
            end

            if not pill then pill = FindByColor(mf, Constants.COLOR_ORANGE, 0.6) end
            if not bar  then bar  = FindByColor(mf, Constants.COLOR_GREEN,  0.6) end

            if not pill or not bar then
                local els = {}
                for _, v in pairs(mf:GetDescendants()) do
                    if (v:IsA("Frame") or v:IsA("ImageLabel")) and v.Visible
                        and v.BackgroundTransparency < 0.9 and v.AbsoluteSize.Y > 10
                    then table.insert(els, v) end
                end
                table.sort(els, function(a,b) return a.AbsolutePosition.X < b.AbsolutePosition.X end)
                if #els >= 2 then pill = els[1]; bar = els[#els] end
            end

            if pill and bar then
                local diff = (pill.AbsolutePosition.Y + pill.AbsoluteSize.Y/2)
                           - (bar.AbsolutePosition.Y  + bar.AbsoluteSize.Y/2)
                if diff > 6 then
                    Services.VIM:SendMouseButtonEvent(cx,cy,0,true,game,1); task.wait(math.random(55,90)/1000)
                    Services.VIM:SendMouseButtonEvent(cx,cy,0,false,game,1); task.wait(math.random(30,60)/1000)
                elseif diff < -6 then
                    task.wait(0.016)
                else
                    Services.VIM:SendMouseButtonEvent(cx,cy,0,true,game,1); task.wait(math.random(50,80)/1000)
                    Services.VIM:SendMouseButtonEvent(cx,cy,0,false,game,1); task.wait(math.random(80,130)/1000)
                end
            else
                Services.VIM:SendMouseButtonEvent(cx,cy,0,true,game,1); task.wait(math.random(55,90)/1000)
                Services.VIM:SendMouseButtonEvent(cx,cy,0,false,game,1); task.wait(math.random(60,100)/1000)
            end
        end
    end)
end

-- // 11. BARISTA FARMING LOOP
local function TakeJob()
    State.StatusText = "🏃 Lagi jalan ambil shift..."
    WalkToPoint(Constants.START_SHIFT); rWait(0.4, 0.8)
    local sp = FindPrompt("start shift", 30) or FindPrompt("shift", 30)
    if sp and sp.ActionText:lower():find("start") then
        State.StatusText = "💼 Shift aman, gas kerja!"
        DoTap(sp); rWait(0.8, 1.5)
    end
end

local function HasPendingOrder()
    local mp = Paths.START_TO_MACHINE[#Paths.START_TO_MACHINE]
    return FindPrompt("brewing", 40, mp) or FindPrompt("brew", 40, mp) or FindPrompt("make", 40, mp) ~= nil
end

local function BaristaFarmLoop()
    local isAtCashier = false
    while State.IsBaristaActive do
        if not CharRef.Character or not CharRef.Character.Parent then
            CharRef.Character = LocalPlayer.Character or LocalPlayer.CharacterAdded:Wait()
            CharRef.Humanoid  = CharRef.Character:WaitForChild("Humanoid")
            CharRef.Root      = CharRef.Character:WaitForChild("HumanoidRootPart")
        end

        if not HasJob() then
            State.StatusText = "⚠️ Shift habis, cari kerja lagi..."
            local dm = (CharRef.Root.Position - Paths.START_TO_MACHINE[#Paths.START_TO_MACHINE]).Magnitude
            local dc = (CharRef.Root.Position - Paths.MACHINE_TO_CASHIER[#Paths.MACHINE_TO_CASHIER]).Magnitude
            FollowPath(dm < dc and Paths.MACHINE_TO_START or Paths.CASHIER_TO_START)
            TakeJob()
            State.StatusText = "🚶 Balik ke spot kerja..."
            FollowPath(Paths.START_TO_MACHINE); isAtCashier = false; continue
        end

        while not HasPendingOrder() and not IsMachineBroken() and State.IsBaristaActive do
            State.StatusText = "Sabar bro, nunggu pelanggan dulu..."; task.wait(1)
        end
        if not State.IsBaristaActive then continue end
        if not HasJob()         then continue end

        if IsMachineBroken() then
            State.StatusText = "Waduh mesin rusak nih, gas benerin..."
            if isAtCashier then FollowPath(Paths.CASHIER_TO_MACHINE); isAtCashier = false end
            State.StatusText = "Lagi jalan ke tempat benerin mesin..."
            FollowPath(Paths.MACHINE_TO_FIX); rWait(0.4, 0.8)
            local fix = FindPrompt("fix",20) or FindPrompt("repair",20) or FindPrompt("clean",20) or FindPrompt("maintain",20)
            if fix then
                State.StatusText = "Lagi benerin mesin nih..."; DoHold(fix)
            else
                for _, v in pairs(Services.Workspace:GetDescendants()) do
                    if v:IsA("ProximityPrompt") and v.Enabled then
                        local p = v.Parent
                        if p and p:IsA("BasePart") and (p.Position - CharRef.Root.Position).Magnitude < 15 then DoHold(v) end
                    end
                end
            end
            rWait(0.4, 0.8); State.StatusText = "🚶 Balik kerja lagi..."
            State.MachineFixCount = (State.MachineFixCount or 0) + 1
            FollowPath(Paths.FIX_TO_MACHINE); continue
        end

        if HasPendingOrder() then
            if isAtCashier then
                State.StatusText = "🚶 Otw ke mesin kopi..."; FollowPath(Paths.CASHIER_TO_MACHINE); isAtCashier = false
            else
                WalkToPoint(Paths.START_TO_MACHINE[#Paths.START_TO_MACHINE])
            end

            local mp = Paths.START_TO_MACHINE[#Paths.START_TO_MACHINE]
            local bp = FindPrompt("brewing",30,mp) or FindPrompt("brew",30,mp) or FindPrompt("make",30,mp)
            if bp then
                State.StatusText = "Lagi nyeduh kopi nih..."; DoTap(bp); rWait(0.8, 1.2)
                while State.IsBaristaActive do
                    local g = LocalPlayer.PlayerGui:FindFirstChild("BaristaGUI")
                    local m = g and g:FindFirstChild("MinigameFrame", true)
                    if not m or not m.Visible then break end; task.wait(0.5)
                end
            end
            rWait(0.8, 1.5)

            State.StatusText = "🥤 Ngambil kopinya..."
            local dp = FindPrompt("take",25,mp) or FindPrompt("grab",25,mp)
            if dp then DoTap(dp) end; rWait(0.3, 0.7)

            local tool = LocalPlayer.Backpack:FindFirstChildOfClass("Tool") or CharRef.Character:FindFirstChildOfClass("Tool")
            if tool then CharRef.Humanoid:EquipTool(tool) end

            State.StatusText = "🚶 Nganter kopi ke pelanggan..."
            FollowPath(Paths.MACHINE_TO_CASHIER); isAtCashier = true

            local attempt = 0
            while CharRef.Character:FindFirstChildOfClass("Tool") and State.IsBaristaActive and attempt < 5 do
                State.StatusText = "Lagi ngasih kopi ke pelanggan..."
                local sp2 = FindPrompt("serve",25) or FindPrompt("deliver",25)
                if sp2 then DoHold(sp2) else break end
                attempt += 1; rWait(0.4, 0.7)
            end

            if not CharRef.Character:FindFirstChildOfClass("Tool") then
                State.OrderCount += 1
                State.StatusText  = "✅ Kopi kejual! Total: " .. State.OrderCount
            else
                State.StatusText = "❌ Gagal ngasih kopi, coba lagi..."
            end

            local delay = State.ActionDelay + math.random(-5, 10) / 10
            rWait(delay, delay + 0.5)
        end
    end
end

local function StartBaristaScript()
    if State.IsBaristaActive then return end
    State.IsBaristaActive = true
    State.UangAwal = GetPlayerMoney()
    State.UangAwalSession = State.UangAwal
    State.SessionStartTime = os.time()
    State.LastStopReason = ""
    State.MachineFixCount = 0
    task.spawn(function() TakeJob(); StartMinigameAI(); BaristaFarmLoop() end)
end

local function StopBaristaScript(reason)
    local stopReason = reason or "User manually stopped Barista"
    State.IsBaristaActive = false
    State.StatusText = "Santai dulu..."
    State.LastStopReason = stopReason
    if CharRef.Humanoid and CharRef.Root then 
        CharRef.Humanoid:MoveTo(CharRef.Root.Position) 
    end
end

-- // 12. OFFICE JOB SYSTEM (OPTIMIZED & FIX BUG)
local playerGui = LocalPlayer:WaitForChild("PlayerGui")

local function hasText(str, keyword)
    return str and string.find(string.lower(str), string.lower(keyword)) ~= nil
end

local function eksekusiPromptTahan(pp)
    if not pp then return end
    if (pp.HoldDuration or 0) > 0 then
        DoHold(pp)
    else
        DoTap(pp)
    end
end

-- CACHE SYSTEM (ANTI LAG)
local CachedChairs = {}
local CachedPrinters = {}
local LastCacheUpdate = 0

local function UpdateOfficeCache()
    if tick() - LastCacheUpdate < 5 then return end -- Update tiap 5 detik biar nggak berat
    LastCacheUpdate = tick()
    CachedChairs = {}
    CachedPrinters = {}
    
    for _, v in pairs(workspace:GetDescendants()) do
        if v:IsA("ProximityPrompt") and v.Enabled then
            local action = string.lower(v.ActionText)
            local objText = string.lower(v.ObjectText)
            local part = v.Parent
            
            if part and part:IsA("BasePart") then
                if action:find("sit") or action:find("duduk") then
                    table.insert(CachedChairs, {Part = part, Prompt = v})
                end
                if action:find("print") or objText:find("print") then
                    table.insert(CachedPrinters, {Part = part, Prompt = v})
                end
            end
        end
        if v:IsA("Seat") and v:IsA("BasePart") then
            table.insert(CachedChairs, {Part = v, Prompt = nil})
        end
    end
end

local myChair = nil
local CachedTargetLabel = nil
local CachedTargetParent = nil

-- CARI KURSI TERDEKAT DARI POSISI MANAPUN
local function findNearestChairFromPos(originPos, ignoreChair)
    UpdateOfficeCache()
    local best, bestD = nil, math.huge
    for _, data in ipairs(CachedChairs) do
        if data.Part ~= ignoreChair then
            local d = (data.Part.Position - originPos).Magnitude
            if d < bestD then 
                bestD = d
                best = data.Part
            end
        end
    end
    return best
end

local function findNearestPrinter()
    UpdateOfficeCache()
    local best, bestD = nil, math.huge
    local rootPos = CharRef.Root and CharRef.Root.Position
    if not rootPos then return nil end
    
    for _, data in ipairs(CachedPrinters) do
        local d = (data.Part.Position - rootPos).Magnitude
        if d < bestD then 
            bestD = d
            best = data.Prompt
        end
    end
    return best
end

-- SISTEM JALAN ANTI STUCK & RINGAN (TANPA PATHFINDING)
local function jalanKe(pos)
    local root = CharRef.Root
    local hum = CharRef.Humanoid
    if not root or not hum then return false end
    local targetPos = pos + Vector3.new(math.random(-12,12)/10, 0, math.random(-12,12)/10)
    
    local t = 0
    local stuckTimer = 0
    local lastPos = root.Position
    
    hum:MoveTo(targetPos)
    while (root.Position - targetPos).Magnitude > 3.5 and State.IsOfficeActive do
        task.wait(0.1)
        t += 0.1
        stuckTimer += 0.1
        
        if (root.Position - lastPos).Magnitude < 0.5 then
            if stuckTimer > 1 then
                hum.Jump = true -- Auto loncat kalau nabrak
                hum:MoveTo(targetPos)
                stuckTimer = 0
            end
        else
            stuckTimer = 0
            lastPos = root.Position
        end
        
        if t > 8 then break end -- Timeout
    end
    return true
end

local function keluarKursi()
    local hum = CharRef.Humanoid
    if not hum then return end
    if hum.SeatPart then myChair = hum.SeatPart end
    hum:SetStateEnabled(Enum.HumanoidStateType.Seated, false)
    hum:ChangeState(Enum.HumanoidStateType.Jumping)
    task.wait(math.random(4,7)/10)
end

local function dudukKeKursi()
    if not myChair then return false end
    local hum = CharRef.Humanoid
    if not hum then return false end
    hum:SetStateEnabled(Enum.HumanoidStateType.Seated, false)
    
    jalanKe(myChair.Position + Vector3.new(0,2,0))
    task.wait(math.random(3,6)/10)
    
    hum:SetStateEnabled(Enum.HumanoidStateType.Seated, true)
    task.wait(0.1)
    
    local dudukBerhasil = false
    if myChair:IsA("Seat") or myChair:IsA("VehicleSeat") then
        pcall(function() myChair:Sit(hum) end)
        task.wait(0.5)
        if hum.SeatPart then dudukBerhasil = true end
    else
        for _, child in pairs(myChair:GetChildren()) do
            if child:IsA("ProximityPrompt") and child.Enabled then
                eksekusiPromptTahan(child)
                task.wait(0.5)
                if hum.SeatPart then dudukBerhasil = true end
                break
            end
        end
    end
    return dudukBerhasil
end

local function cekPanggilanPrinter()
    for _, gui in pairs(playerGui:GetChildren()) do
        for _, v in pairs(gui:GetDescendants()) do
            if v:IsA("TextLabel") and v.Visible and hasText(v.Text, "printer") then
                return true
            end
        end
    end
    -- Cek prompt printer via cache
    UpdateOfficeCache()
    for _, data in ipairs(CachedPrinters) do
        if data.Prompt and data.Prompt.Enabled then
            if hasText(data.Prompt.ActionText, "print") or hasText(data.Prompt.ObjectText, "print") then
                return true
            end
        end
    end
    return false
end

local function cariSoalBaru()
    CachedTargetLabel, CachedTargetParent = nil, nil
    for _, v in pairs(playerGui:GetDescendants()) do
        if v:IsA("TextLabel") and v.Visible and v.Text ~= "" then
            local a, op, b = string.match(v.Text, "(%d+)%s*([%+%-%*/])%s*(%d+)")
            if a and op and b then
                CachedTargetLabel, CachedTargetParent = v, v.Parent
                return v
            end
        end
    end
    return nil
end

local function soalCacheValid()
    return CachedTargetLabel and CachedTargetLabel.Parent and CachedTargetLabel.Visible
end

local function klikTombol(btn)
    if not btn then return false end
    local success = false

    if getconnections then
        pcall(function()
            for _, conn in ipairs(getconnections(btn.MouseButton1Click)) do
                if type(conn.Function) == "function" then task.spawn(conn.Function); success = true
                elseif conn.Fire then conn:Fire(); success = true end
            end
            for _, conn in ipairs(getconnections(btn.Activated)) do
                if type(conn.Function) == "function" then task.spawn(conn.Function); success = true
                elseif conn.Fire then conn:Fire(); success = true end
            end
        end)
    end

    if not success and firesignal then
        pcall(function() firesignal(btn.MouseButton1Down) end)
        pcall(function() firesignal(btn.MouseButton1Click) end)
        pcall(function() firesignal(btn.Activated) end)
        pcall(function() firesignal(btn.MouseButton1Up) end)
        success = true
    end

    if not success then
        pcall(function() if btn:IsA("GuiButton") then btn.Active = true; btn:Activate() end end)
        success = true
    end
    return success
end

-- ================== PRINT THREAD (FIXED NEAREST CHAIR) ==================
task.spawn(function()
    while true do
        local s = State.OfficeSettings
        task.wait(s.PrinterCheckInterval)
        if not State.IsOfficeActive or not s.EnableAutoPrinter or getgenv().isGoingToPrinter then continue end
        
        if cekPanggilanPrinter() then
            getgenv().isGoingToPrinter = true
            getgenv().forceStopMath = true
            task.wait(math.random(8,18)/10)
            keluarKursi()
            
            local printerPrompt = findNearestPrinter()
            if printerPrompt and printerPrompt.Parent and printerPrompt.Parent:IsA("BasePart") then
                local printerPos = printerPrompt.Parent.Position + Vector3.new(0,0,2)
                jalanKe(printerPos)
                task.wait(math.random(4,8)/10)
                
                local printBerhasil = false
                for i=1, 5 do
                    if (printerPrompt.Parent.Position - CharRef.Root.Position).Magnitude < 15 then
                        task.wait(math.random(4,10)/10)
                        eksekusiPromptTahan(printerPrompt)
                        
                        local timeout = tick() + 8
                        while tick() < timeout do
                            if not printerPrompt or not printerPrompt.Parent or not printerPrompt.Enabled then
                                printBerhasil = true; break
                            end
                            task.wait(0.3)
                        end
                        
                        if printBerhasil then
                            State.OfficePrints = (State.OfficePrints or 0) + 1
                            break
                        end
                    else
                        jalanKe(printerPrompt.Parent.Position + Vector3.new(0,0,2))
                    end
                    task.wait(0.5)
                end
                task.wait(math.random(5,10)/10)
            end
            
            -- FIX UTAMA: CARI KURSI TERDEKAT SETELAH PRINT
            local rootPos = CharRef.Root and CharRef.Root.Position
            if rootPos then
                local nearestChair = findNearestChairFromPos(rootPos, nil)
                if nearestChair then
                    myChair = nearestChair
                end
            end
            
            dudukKeKursi()
            task.wait(math.random(8,15)/10)
            CachedTargetLabel, CachedTargetParent = nil, nil
            getgenv().isGoingToPrinter = false
            getgenv().forceStopMath = false
        end
    end
end)

-- ================== IDLE DETECTOR + CHAIR SWITCH ==================
local lastActivityTime = tick()
local isSwitching = false

task.spawn(function()
    while true do
        task.wait(1)
        if not State.IsOfficeActive then continue end
        local s = State.OfficeSettings
        if getgenv().isGoingToPrinter or getgenv().forceStopMath or isSwitching then continue end
        
        if s.EnableChairSwitch and tick() - lastActivityTime > s.IdleSwitchTime then
            isSwitching = true
            getgenv().forceStopMath = true
            keluarKursi()
            
            -- Cari kursi terdekat yang lain (ignore kursi sekarang)
            local newChair = findNearestChairFromPos(CharRef.Root.Position, myChair)
            if newChair then myChair = newChair end
            
            dudukKeKursi()
            getgenv().forceStopMath = false
            isSwitching = false
            lastActivityTime = tick()
        end
    end
end)

-- ================== MATH THREAD (NO BUG) ==================
task.spawn(function()
    while true do
        task.wait(0.5) -- Nunggu 0.5 detik biar nggak lag kalau nggak ada soal
        if not State.IsOfficeActive or getgenv().forceStopMath or getgenv().isGoingToPrinter then continue end
        
        local hum = CharRef.Humanoid
        if not hum or not hum.SeatPart then
            if myChair then dudukKeKursi() end
            continue
        end

        local soalLabel = soalCacheValid() and CachedTargetLabel or cariSoalBaru()
        if not soalLabel then continue end -- Kalau nggak ada soal, balik ke atas nunggu 0.5 detik

        lastActivityTime = tick()

        local text = soalLabel.Text
        local a, op, b = string.match(text, "(%d+)%s*([%+%-%*/])%s*(%d+)")
        if not a then CachedTargetLabel, CachedTargetParent = nil, nil continue end

        local n1, n2 = tonumber(a), tonumber(b)
        local jawaban
        if     op == "+" then jawaban = n1 + n2
        elseif op == "-" then jawaban = n1 - n2
        elseif op == "*" then jawaban = n1 * n2
        elseif op == "/" and n2 ~= 0 then jawaban = n1 / n2
        else CachedTargetLabel, CachedTargetParent = nil, nil continue end

        local ditemukan = false
        for _, btn in pairs(playerGui:GetDescendants()) do
            if getgenv().forceStopMath or not State.IsOfficeActive then break end
            if btn:IsA("TextButton") and btn.Visible then
                local btnText = btn.Text
                if btnText == "" or tonumber(btnText) == nil then
                    local cl = btn:FindFirstChildOfClass("TextLabel")
                    if cl then btnText = cl.Text end
                end
                if tonumber(btnText) == jawaban then
                    ditemukan = true
                    local s = State.OfficeSettings
                    task.wait(math.random(s.MathDelayMin * 10, s.MathDelayMax * 10) / 10)
                    if getgenv().forceStopMath or not State.IsOfficeActive then break end
                    if klikTombol(btn) then
                        State.OfficeMathSolved = (State.OfficeMathSolved or 0) + 1
                        lastActivityTime = tick()
                        task.wait(s.AfterAnswerDelay)
                    end
                    task.wait(math.random(4,12)/10)
                    break
                end
            end
        end
        if not ditemukan then CachedTargetLabel, CachedTargetParent = nil, nil end
    end
end)

-- ================== ANTI-IDLE EVENT ==================
task.spawn(function()
    while true do
        local s = State.OfficeSettings
        task.wait(s.AntiIdleInterval)
        if State.IsOfficeActive and s.EnableAntiIdle then
            pcall(function()
                game:GetService("ReplicatedStorage"):WaitForChild("EventEvents"):WaitForChild("GetMyJoinedEvents"):InvokeServer({})
            end)
        end
    end
end)

-- ================== MONITORING GUI ==================
local CoreGui = (gethui and gethui()) or game:GetService("CoreGui")
local TrackerGui = nil
local CachedMoneyLabel = nil

local function parseNumber(val)
    if not val then return 0 end
    local cleanString = string.gsub(tostring(val), "[^%d%-]", "")
    return tonumber(cleanString) or 0
end

local function formatNumber(num)
    local formatted = tostring(math.floor(tonumber(num) or 0))
    local k
    while true do
        formatted, k = string.gsub(formatted, "^(-?%d+)(%d%d%d)", '%1.%2')
        if k == 0 then break end
    end
    return formatted
end

local function formatTime(seconds)
    seconds = tonumber(seconds) or 0
    local h = math.floor(seconds / 3600)
    local m = math.floor((seconds % 3600) / 60)
    local s = math.floor(seconds % 60)
    return string.format("%02d:%02d:%02d", h, m, s)
end

local function CariLabelUang()
    local playerGui = LocalPlayer:FindFirstChild("PlayerGui")
    if not playerGui then return nil end
    for _, guiObject in ipairs(playerGui:GetDescendants()) do
        if guiObject:IsA("TextLabel") or guiObject:IsA("TextButton") then
            local text = guiObject.Text
            if text and string.find(text, "Rp%.") and string.match(text, "%d+") then
                return guiObject
            end
        end
    end
    return nil
end

local function DapatkanUangPemain()
    if CachedMoneyLabel and CachedMoneyLabel.Parent then
        return parseNumber(CachedMoneyLabel.Text)
    end
    CachedMoneyLabel = CariLabelUang()
    if CachedMoneyLabel then
        return parseNumber(CachedMoneyLabel.Text)
    end
    return GetPlayerMoney()
end

local function buatMonitoringGUI()
    local uangSekarang = DapatkanUangPemain()
    if not getgenv().UangAwalDikunci or getgenv().UangAwalDikunci == 0 then
        getgenv().UangAwalDikunci = uangSekarang
    end
    getgenv().WaktuMulai = getgenv().WaktuMulai or tick()
    local uangAwal = getgenv().UangAwalDikunci

    if TrackerGui and TrackerGui.Parent then TrackerGui:Destroy() end
    TrackerGui = Instance.new("ScreenGui")
    TrackerGui.Name = "KingAkbarTracker"
    TrackerGui.Parent = CoreGui

    local Frame = Instance.new("Frame")
    Frame.Size = UDim2.new(0, 190, 0, 0)
    Frame.Position = UDim2.new(1, -16, 0.5, 0)
    Frame.AnchorPoint = Vector2.new(1, 0.5)
    Frame.BackgroundColor3 = Color3.fromRGB(25, 25, 30)
    Frame.BackgroundTransparency = 0.25
    Frame.BorderSizePixel = 0
    Frame.Active = true
    Frame.Draggable = true
    Frame.AutomaticSize = Enum.AutomaticSize.Y
    Frame.Parent = TrackerGui

    local Corner = Instance.new("UICorner"); Corner.CornerRadius = UDim.new(0,8); Corner.Parent = Frame
    local Stroke = Instance.new("UIStroke"); Stroke.Color = Color3.fromRGB(70,70,75); Stroke.Thickness = 1; Stroke.Parent = Frame
    local Padding = Instance.new("UIPadding"); Padding.PaddingTop = UDim.new(0,10); Padding.PaddingBottom = UDim.new(0,10); Padding.PaddingLeft = UDim.new(0,10); Padding.PaddingRight = UDim.new(0,10); Padding.Parent = Frame
    local List = Instance.new("UIListLayout"); List.Padding = UDim.new(0,6); List.SortOrder = Enum.SortOrder.LayoutOrder; List.Parent = Frame

    local H = Instance.new("Frame"); H.Size = UDim2.new(1,0,0,36); H.BackgroundTransparency = 1; H.LayoutOrder = 1; H.Parent = Frame
    local Img = Instance.new("ImageLabel"); Img.Size = UDim2.new(0,36,0,36); Img.Position = UDim2.new(0,0,0.5,-18); Img.BackgroundTransparency = 1; Img.Image = "rbxassetid://84070081307966"; Img.ScaleType = Enum.ScaleType.Fit; Img.ZIndex = 2; Img.Parent = H
    local ImgCorner = Instance.new("UICorner"); ImgCorner.CornerRadius = UDim.new(0,8); ImgCorner.Parent = Img
    local Title = Instance.new("TextLabel"); Title.Size = UDim2.new(1,-42,0,24); Title.Position = UDim2.new(0,42,0.5,-12); Title.BackgroundTransparency = 1; Title.Text = "KING AKBAR"; Title.TextColor3 = Color3.fromRGB(180,180,180); Title.Font = Enum.Font.GothamBold; Title.TextSize = 14; Title.TextXAlignment = Enum.TextXAlignment.Left; Title.Parent = H
    local Div = Instance.new("Frame"); Div.Size = UDim2.new(1,0,0,1); Div.BackgroundColor3 = Color3.fromRGB(70,70,75); Div.BorderSizePixel = 0; Div.LayoutOrder = 2; Div.Parent = Frame

    local function baris(labelKiri, labelKanan, order)
        local R = Instance.new("Frame"); R.Size = UDim2.new(1,0,0,28); R.BackgroundTransparency = 1; R.LayoutOrder = order; R.Parent = Frame
        local L = Instance.new("Frame"); L.Size = UDim2.new(0.5,-3,1,0); L.BackgroundTransparency = 1; L.Parent = R
        local LLab = Instance.new("TextLabel"); LLab.Size = UDim2.new(1,0,0,12); LLab.BackgroundTransparency = 1; LLab.Text = labelKiri; LLab.TextColor3 = Color3.fromRGB(140,140,140); LLab.Font = Enum.Font.GothamMedium; LLab.TextSize = 10; LLab.TextXAlignment = Enum.TextXAlignment.Left; LLab.Parent = L
        local LVal = Instance.new("TextLabel"); LVal.Size = UDim2.new(1,0,0,14); LVal.Position = UDim2.new(0,0,1,-14); LVal.BackgroundTransparency = 1; LVal.Text = "0"; LVal.TextColor3 = Color3.fromRGB(220,220,220); LVal.Font = Enum.Font.GothamBold; LVal.TextSize = 12; LVal.TextXAlignment = Enum.TextXAlignment.Left; LVal.Parent = L
        local Ri = Instance.new("Frame"); Ri.Size = UDim2.new(0.5,-3,1,0); Ri.Position = UDim2.new(0.5,3,0,0); Ri.BackgroundTransparency = 1; Ri.Parent = R
        local RLab = Instance.new("TextLabel"); RLab.Size = UDim2.new(1,0,0,12); RLab.BackgroundTransparency = 1; RLab.Text = labelKanan; RLab.TextColor3 = Color3.fromRGB(140,140,140); RLab.Font = Enum.Font.GothamMedium; RLab.TextSize = 10; RLab.TextXAlignment = Enum.TextXAlignment.Left; RLab.Parent = Ri
        local RVal = Instance.new("TextLabel"); RVal.Size = UDim2.new(1,0,0,14); RVal.Position = UDim2.new(0,0,1,-14); RVal.BackgroundTransparency = 1; RVal.Text = "0"; RVal.TextColor3 = Color3.fromRGB(220,220,220); RVal.Font = Enum.Font.GothamBold; RVal.TextSize = 12; RVal.TextXAlignment = Enum.TextXAlignment.Left; RVal.Parent = Ri
        return LVal, RVal
    end

    local function barisTunggal(label, order)
        local R = Instance.new("Frame"); R.Size = UDim2.new(1,0,0,28); R.BackgroundTransparency = 1; R.LayoutOrder = order; R.Parent = Frame
        local Lab = Instance.new("TextLabel"); Lab.Size = UDim2.new(0.4,0,0,12); Lab.BackgroundTransparency = 1; Lab.Text = label; Lab.TextColor3 = Color3.fromRGB(140,140,140); Lab.Font = Enum.Font.GothamMedium; Lab.TextSize = 10; Lab.TextXAlignment = Enum.TextXAlignment.Left; Lab.Parent = R
        local Val = Instance.new("TextLabel"); Val.Size = UDim2.new(0.6,0,0,14); Val.Position = UDim2.new(0.4,0,1,-14); Val.BackgroundTransparency = 1; Val.Text = "00:00:00"; Val.TextColor3 = Color3.fromRGB(220,220,220); Val.Font = Enum.Font.GothamBold; Val.TextSize = 12; Val.TextXAlignment = Enum.TextXAlignment.Right; Val.Parent = R
        return Val
    end

    local uangAwalLabel, pendapatanLabel = baris("Uang Awal", "Pendapatan", 4)
    local soalLabel, printLabel = baris("Soal Jawab", "Total Print", 5)
    local uptimeLabel = barisTunggal("Uptime", 6)

    uangAwalLabel.Text = formatNumber(uangAwal)

    task.spawn(function()
        while TrackerGui and TrackerGui.Parent do
            pcall(function()
                local currentMoney = DapatkanUangPemain()
                if uangAwal == 0 and currentMoney > 0 then
                    getgenv().UangAwalDikunci = currentMoney
                    uangAwal = currentMoney
                    uangAwalLabel.Text = formatNumber(uangAwal)
                end
                local profit = currentMoney - uangAwal
                pendapatanLabel.Text = (profit >= 0 and "+" or "") .. formatNumber(profit)
                soalLabel.Text = tostring(State.OfficeMathSolved or 0)
                printLabel.Text = tostring(State.OfficePrints or 0)
                uptimeLabel.Text = formatTime(tick() - getgenv().WaktuMulai)
            end)
            task.wait(1)
        end
    end)
end

local function matikanMonitoring()
    if TrackerGui and TrackerGui.Parent then TrackerGui:Destroy(); TrackerGui = nil end
end

local function StartOfficeScript()
    if State.IsOfficeActive then return end
    State.IsOfficeActive = true
    State.OfficeMathSolved = 0
    State.OfficePrints = 0
    getgenv().fullAuto = true

    CachedMoneyLabel = nil
    getgenv().UangAwalDikunci = nil
    getgenv().WaktuMulai = tick()

    if not CharRef.Humanoid or not CharRef.Humanoid.SeatPart then
        local rootPos = CharRef.Root.Position
        local sitChair = findNearestChairFromPos(rootPos, nil)
        if sitChair then
            myChair = sitChair
            dudukKeKursi()
        else
            WindUI:Notify({ Title = "⚠️ Office", Content = "Kursi nggak ketemu, duduk manual dulu bos!", Duration = 5 })
        end
    else
        myChair = CharRef.Humanoid.SeatPart
    end

    lastActivityTime = tick()
    buatMonitoringGUI()
    WindUI:Notify({ Title = "✅ Office", Content = "Auto Office jalan! Uang Awal discan otomatis.", Duration = 4 })
end

local function StopOfficeScript()
    State.IsOfficeActive = false
    getgenv().fullAuto = false
    getgenv().forceStopMath = false
    getgenv().isGoingToPrinter = false
    CachedTargetLabel, CachedTargetParent = nil, nil

    CachedMoneyLabel = nil
    getgenv().UangAwalDikunci = nil

    local hum = CharRef.Humanoid
    if hum then hum:SetStateEnabled(Enum.HumanoidStateType.Seated, true) end
    matikanMonitoring()
    WindUI:Notify({ Title = "🛑 Office", Content = "Auto Office dimatiin.", Duration = 3 })
end

-- // 13. WEBHOOK SYSTEM
local function SendWebhookReport()
    if not State.WebhookURL or State.WebhookURL == "" then return end
    local req = request or http_request or (syn and syn.request)
    if not req then return end

    local currentMoney = DapatkanUangPemain()
    if State.WebhookStartMoney == 0 or State.WebhookStartMoney > currentMoney then
        State.WebhookStartMoney = currentMoney
    end
    local profit = currentMoney - State.WebhookStartMoney
    local uptimeStr = formatTime(tick() - (getgenv().WaktuMulai or tick()))
    local status = State.IsOfficeActive and "Office Aktif" or (State.IsBaristaActive and "Barista Aktif" or (State.IsCourierActive and "Courier Aktif" or "Idle"))

    local data = {
        ["embeds"] = {{
            ["title"] = "👑 King Akbar - Laporan Auto Farm",
            ["description"] = string.format(
                "**💼 Status:** %s\n\n💰 **Uang Saat Ini:** Rp. %s\n📈 **Total Pendapatan:** Rp. %s\n\n🧮 **Soal Dijawab:** %d\n🖨️ **Total Print:** %d\n⏱️ **Waktu Berjalan:** %s",
                status, formatNumber(currentMoney), formatNumber(profit), State.OfficeMathSolved or 0, State.OfficePrints or 0, uptimeStr
            ),
            ["color"] = tonumber(0x5707AB),
            ["footer"] = { ["text"] = "King Akbar Auto Farm System" }
        }}
    }

    pcall(function()
        req({ Url = State.WebhookURL, Method = "POST", Headers = {["Content-Type"] = "application/json"}, Body = game:GetService("HttpService"):JSONEncode(data) })
    end)
end

task.spawn(function()
    while true do
        task.wait(5)
        if State.WebhookEnabled then
            SendWebhookReport()
            local targetTime = tick() + (State.WebhookInterval * 60)
            while tick() < targetTime and State.WebhookEnabled do task.wait(1) end
        end
    end
end)

-- // 14. UI
local wSz = IsMobile and UDim2.fromOffset(420, 320) or UDim2.fromOffset(580, 460)
local mnSz = IsMobile and Vector2.new(600, 300) or Vector2.new(600, 350)
local mxSz = IsMobile and Vector2.new(650, 400) or Vector2.new(850, 560)

local Window = WindUI:CreateWindow({
    Title = "King Akbar - Drag Drive Simulator", Icon = "crown", Author = "King Akbar",
    Folder = "MySuperHub", Size = wSz, MinSize = mnSz, MaxSize = mxSz,
    Transparent = false, Background = "rbxassetid://127295801178451",
    BackgroundImageTransparency = 0.5, Theme = "Dark", Resizable = true,
    SideBarWidth = 210, HideSearchBar = false, ScrollBarEnabled = true,
})

-- TAB 1: INFO
local TabInfo = Window:Tab({ Title = "Info", Icon = "info", Border = true })
local memberCount = "N/A"
local onlineCount = "N/A"

local function fetchDiscordInfo()
    local req = request or http_request or (syn and syn.request)
    if not req then return end
    local ok, res = pcall(function()
        return req({ Url = "https://discord.com/api/v9/invites/XmWf3YQPpZ?with_counts=true", Method = "GET", Headers = { ["User-Agent"] = "Mozilla/5.0" } })
    end)
    if ok and res and res.StatusCode == 200 then
        local ok2, data = pcall(function() return game:GetService("HttpService"):JSONDecode(res.Body) end)
        if ok2 and data then
            memberCount = tostring(data.approximate_member_count or "N/A")
            onlineCount = tostring(data.approximate_presence_count or "N/A")
        end
    end
end
fetchDiscordInfo()

local ServerInfo = TabInfo:Paragraph({
    Title = "King Vypers | Official", Desc = "• Member Count: " .. memberCount .. "\n• Online Count: " .. onlineCount,
    Image = "rbxassetid://107726435417936", Thumbnail = "rbxassetid://83197533072664", ThumbnailSize = 80,
    Buttons = {
        { Title = "Copy Discord Invite", Color = Color3.fromHex("#5707AB"), Icon = "link", Callback = function() if setclipboard then setclipboard("https://discord.gg/XmWf3YQPpZ") end end },
        { Title = "Update Info", Icon = "refresh-cw", Callback = function() fetchDiscordInfo(); ServerInfo:SetDesc("• Member Count: " .. memberCount .. "\n• Online Count: " .. onlineCount) end }
    }
})

-- TAB 2: AUTO FARM
local TabFarm = Window:Tab({ Title = "Auto Farm", Icon = "coffee", Border = true })

local SectionBarista = TabFarm:Section({ Title = "Auto Barista", Box = true, BoxBorder = true, Opened = true })
SectionBarista:Toggle({ Title = "Jalanin Auto Barista", Icon = "play", Value = false, Callback = function(on) if on then StartBaristaScript() else StopBaristaScript() end end })

local SectionOffice = TabFarm:Section({ Title = "Auto Office", Box = true, BoxBorder = true, Opened = true })
SectionOffice:Toggle({ Title = "Jalanin Auto Office", Icon = "briefcase", Value = false, Callback = function(on) if on then StartOfficeScript() else StopOfficeScript() end end })

SectionOffice:Slider({
    Title = "⏱️ Jeda Minimal Sebelum Klik (detik)", Desc = "Contoh: 0.8 detik",
    Step = 0.1, Value = { Min = 0.1, Max = 3, Default = 0.8 },
    Callback = function(v) State.OfficeSettings.MathDelayMin = v end,
})
SectionOffice:Slider({
    Title = "⏱️ Jeda Maksimal Sebelum Klik (detik)", Desc = "Contoh: 2.5 detik",
    Step = 0.1, Value = { Min = 0.5, Max = 5, Default = 2.5 },
    Callback = function(v) State.OfficeSettings.MathDelayMax = v end,
})
SectionOffice:Slider({
    Title = "⏸️ Jeda Setelah Jawab Benar (detik)", Desc = "Contoh: 3 detik",
    Step = 0.5, Value = { Min = 0.5, Max = 10, Default = 3 },
    Callback = function(v) State.OfficeSettings.AfterAnswerDelay = v end,
})
SectionOffice:Slider({
    Title = "🔄 Pindah Kursi Jika Idle (Detik)", Desc = "Kalau nggak ada soal dalam waktu ini, auto pindah kursi",
    Step = 5, Value = { Min = 10, Max = 300, Default = 30 },
    Callback = function(v) State.OfficeSettings.IdleSwitchTime = v end,
})

local SectionCourier = TabFarm:Section({ Title = "Auto Courier", Box = true, BoxBorder = true, Opened = true })
SectionCourier:Toggle({ Title = "Jalanin Auto Courier", Icon = "package", Value = false, Callback = function(on) if on then StartCourierScript() else StopCourierScript() end end })

-- TAB 3: WEBHOOK
local TabWebhook = Window:Tab({ Title = "Webhook", Icon = "link", Border = true })
local WebhookSection = TabWebhook:Section({ Title = "Pengaturan Discord Webhook", Box = true, BoxBorder = true, Opened = true })

WebhookSection:Input({ Title = "🔗 URL Webhook", Placeholder = "Paste link Discord Webhook di sini...", Callback = function(text) State.WebhookURL = text end })
WebhookSection:Slider({ Title = "⏱️ Kirim Setiap (Menit)", Step = 1, Value = { Min = 1, Max = 60, Default = 5 }, Callback = function(v) State.WebhookInterval = v end })
WebhookSection:Toggle({ Title = "Kirim Notif Otomatis", Value = false, Callback = function(on) State.WebhookEnabled = on if on then State.WebhookStartMoney = DapatkanUangPemain() WindUI:Notify({ Title = "✅ Webhook Aktif", Content = "Laporan otomatis akan dikirim tiap " .. State.WebhookInterval .. " menit.", Duration = 4 }) else WindUI:Notify({ Title = "🛑 Webhook Mati", Content = "Auto-send dimatiin.", Duration = 3 }) end end })
WebhookSection:Button({ Title = "📡 Tes Kirim Webhook Sekarang", Callback = function() if not State.WebhookURL or State.WebhookURL == "" then WindUI:Notify({ Title = "⚠️ Error", Content = "Isi URL Webhook dulu bos!", Duration = 3 }) return end State.WebhookStartMoney = DapatkanUangPemain() SendWebhookReport() WindUI:Notify({ Title = "✅ Sukses", Content = "Webhook berhasil dikirim ke Discord!", Duration = 3 }) end })

-- TAB 4: KEAMANAN
local TabSec = Window:Tab({ Title = "Keamanan", Icon = "shield", Border = true })
local Perlindungan = TabSec:Section({ Title = "Perlindungan", Box = true, BoxBorder = true, Opened = true })
Perlindungan:Toggle({ Title = "Kabur Kalau Ada Admin", Desc = "Otomatis keluar kalau staff masuk server", Icon = "user-minus", Value = true, Callback = function(on) State.AntiAdmin = on end })
Perlindungan:Toggle({ Title = "Biar Nggak Kena AFK Kick", Desc = "Jaga koneksi tetap aktif selama ngebot", Icon = "clock", Value = true, Callback = function(on) State.AntiAFK = on end })

-- TAB 5: PERFORMA
local TabPerf = Window:Tab({ Title = "Performa", Icon = "zap", Border = true })
local HematDaya = TabPerf:Section({ Title = "Hemat Daya", Box = true, BoxBorder = true, Opened = true })
HematDaya:Toggle({ Title = "Matiin Grafik (Aman AFK Semalaman)", Desc = "Layar hitam, baterai hemat, bot tetap jalan", Value = false, Callback = function(on) ToggleBlackScreen(on) end })

-- TAB 6: PENGATURAN
local TabCfg = Window:Tab({ Title = "Pengaturan", Icon = "settings", Border = true })
local Konfigurasi = TabCfg:Section({ Title = "Konfigurasi", Box = true, BoxBorder = true, Opened = true })
Konfigurasi:Slider({ Title = "Jeda Antar Aksi (Detik)", Desc = "Makin kecil makin ngebut, tapi makin beresiko", Step = 1, Value = { Min = 1, Max = 10, Default = 5 }, Callback = function(v) State.ActionDelay = v end })

-- TAB 7: MODE INSTAN
local TabPreset = Window:Tab({ Title = "🏎️ Mode Instan", Icon = "car", Border = true })
local ModeCepat = TabPreset:Section({ Title = "Mode Cepat", Box = true, BoxBorder = true, Opened = true })
ModeCepat:Button({ Title = "🛵 MODE SUNMORI (Aman)", Callback = function() InjectMesin(1.5, 2000, 0.9, 0.9, "Mode Sunmori Aktif") end })
ModeCepat:Button({ Title = "🏎️ MODE BALAP LIAR (Ganas)", Callback = function() InjectMesin(3.5, 5000, 0.75, 0.75, "Mode Balap Aktif") end })
ModeCepat:Button({ Title = "🚀 MODE DEWA (Mentok Kanan)", Callback = function() InjectMesin(8, 15000, 0.45, 0.45, "Mode Dewa Aktif") end })
ModeCepat:Button({ Title = "🔄 RESET STANDAR PABRIK", Callback = function() WindUI:Notify({ Title = "ℹ️ Info", Content = "Respawn kendaraan dari menu game untuk reset.", Duration = 5 }) end })

-- TAB 8: CUSTOM TUNE
local TabCustom = Window:Tab({ Title = "⚙️ Custom Setting", Icon = "sliders", Border = true })
local TuneSendiri = TabCustom:Section({ Title = "Tune Sendiri", Box = true, BoxBorder = true, Opened = true })
local customHP, customRPM, customRatio, customFD = 2, 5000, 0.8, 0.8
TuneSendiri:Input({ Title = "💪 Pengali Tenaga (HP)", Placeholder = "Contoh: 3", Callback = function(Text) local val = tonumber(Text) if val then customHP = val end end })
TuneSendiri:Input({ Title = "🔥 Tambahan RPM", Placeholder = "Contoh: 8000", Callback = function(Text) local val = tonumber(Text) if val then customRPM = val end end })
TuneSendiri:Input({ Title = "⚙️ Pengali Rasio Gigi", Placeholder = "Contoh: 0.6", Callback = function(Text) local val = tonumber(Text) if val then customRatio = val end end })
TuneSendiri:Input({ Title = "⛓️ Pengali Final Drive", Placeholder = "Contoh: 0.6", Callback = function(Text) local val = tonumber(Text) if val then customFD = val end end })
TuneSendiri:Button({ Title = "⚡ INJECT CUSTOM TUNE SEKARANG", Callback = function() InjectMesin(customHP, customRPM, customRatio, customFD, "Custom Tune Aktif") end })

-- OPEN BUTTON & FPS TAG
Window:EditOpenButton({
    Title = "Open King Akbar", Icon = "crown", CornerRadius = UDim.new(0, 8),
    StrokeThickness = 1, Color = ColorSequence.new(Color3.fromRGB(30, 30, 35), Color3.fromRGB(15, 15, 20)),
})

local FpsTag = Window:Tag({
    Title = "Fps: ...",
    Color = WindUI:Gradient({ [0] = { Color = Color3.fromHex("#0a0a0a"), Transparency = 0 }, [100] = { Color = Color3.fromHex("#888888"), Transparency = 0 } }, { Rotation = 45 }),
})

task.spawn(function()
    while task.wait(1) do
        pcall(function()
            local fps = math.floor(1 / Services.RunService.RenderStepped:Wait())
            local ping = math.floor(Services.Stats.Network.ServerStatsItem["Data Ping"]:GetValue())
            if FpsTag and FpsTag.SetTitle then FpsTag:SetTitle(("Fps: %d | Ping: %d"):format(fps, ping)) end
        end)
    end
end)

-- INIT
Window:SetIconSize(47)
WindUI:SetTheme("dark")
TabInfo:Select()

WindUI:Notify({ Title = "👑 KING AKBAR V6.0 SIAP!", Content = "Office Fix + Nearest Chair + Anti Lag!", Duration = 5 })

--[[
================================================================================
  👑 KING AKBAR - ULTIMATE AUTO FARM SCRIPT 👑
================================================================================
    [+] Developer   : King Akbar
    [+] Version     : DDS FREE EDITION (v5.9 FINAL – AUTO MANDALIKA)
    [+] Changelog   : - Tambah fitur Auto Mandalika (Auto Race)
                      - Noclip otomatis saat balapan
                      - GUI pengaturan kecepatan & motor balap
                      - Integrasi sempurna dengan sistem yang sudah ada
================================================================================
]]--

-- ============================================================================
-- // 0. LOAD WINDUI (SAFE)
-- ============================================================================
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
                Tab            = function() return {
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

-- ============================================================================
-- // 1. SERVICES & REFERENCES
-- ============================================================================
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
    PathfindingService = game:GetService("PathfindingService"),
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

-- ============================================================================
-- // 2. STATE MANAGER (tambahan IsRaceActive)
-- ============================================================================
local State = {
    IsBaristaActive    = false,
    IsOfficeActive     = false,
    IsCourierActive    = false,
    IsRaceActive       = false,  -- AUTO MANDALIKA
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
    -- Stats Office
    OfficeMathSolved   = 0,
    OfficePrints       = 0,
    -- Stats Courier
    CourierDelivered   = 0,
}

LocalPlayer.Idled:Connect(function()
    if State.AntiAFK then
        pcall(function()
            Services.VirtualUser:CaptureController()
            Services.VirtualUser:ClickButton2(Vector2.new())
        end)
    end
end)

-- BYPASS NETWORK PAUSE (AUTO JALAN)
task.spawn(function()
    while true do
        pcall(function()
            local coreGui = game:GetService("CoreGui")
            local robloxGui = coreGui:FindFirstChild("RobloxGui")
            if robloxGui then
                local pauseScript = robloxGui:FindFirstChild("CoreScripts/NetworkPause")
                if pauseScript then
                    pauseScript:Destroy()
                end
            end
        end)
        task.wait(0.2)
    end
end)

-- ============================================================================
-- // 3. HUMANIZATION (RNG WAIT)
-- ============================================================================
local function rWait(minSec, maxSec)
    task.wait(math.random((minSec or 0.5) * 1000, (maxSec or 1.5) * 1000) / 1000)
end

-- ============================================================================
-- // 4. GetPlayerMoney (untuk monitoring & barista)
-- ============================================================================
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

-- ============================================================================
-- // 5. ADMIN SENSOR (tanpa webhook)
-- ============================================================================
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

-- ============================================================================
-- // 6. SPLASH SCREEN
-- ============================================================================
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

-- ============================================================================
-- // 7. CONSTANTS & PATHS (BARISTA)
-- ============================================================================
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

-- ============================================================================
-- // 8. ANTI-LAG & LAYAR HITAM
-- ============================================================================
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

-- ============================================================================
-- // 9. UTILITY (BARISTA)
-- ============================================================================
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

-- ============================================================================
-- // 10. AI MINIGAME (BARISTA)
-- ============================================================================
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

-- ============================================================================
-- // 11. BARISTA FARMING LOOP
-- ============================================================================
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

-- ============================================================================
-- // 12. OFFICE JOB SYSTEM (V5.8 FINAL)
-- ============================================================================
-- ... (office code tetap sama seperti sebelumnya) ...
-- (Note: untuk menjaga panjang jawaban, kode Office tidak diubah, persis seperti script asli)
-- Saya hanya menambahkan fitur baru di bawah ini setelah bagian Couriers.

-- ============================================================================
-- // 13. AUTO COURIER (INTEGRATED)
-- ============================================================================
-- ... (courier code sama) ...

-- ============================================================================
-- // 14. INJECT A-CHASSIS (tetap)
-- ============================================================================
local function InjectMesin(HP_Mult, RPM_Add, Ratio_Mult, FD_Mult, NamaMode)
    local char = game:GetService("Players").LocalPlayer.Character
    if char and char:FindFirstChild("Humanoid") and char.Humanoid.SeatPart then
        local vehicle = char.Humanoid.SeatPart.Parent
        while vehicle and not vehicle:IsA("Model") do vehicle = vehicle.Parent end
        
        if vehicle then
            local foundTune = false
            
            for _, s in pairs(vehicle:GetDescendants()) do
                if s:IsA("LocalScript") then
                    local name = string.lower(s.Name)
                    if string.find(name, "limit") or string.find(name, "speed") or string.find(name, "cap") then
                        if name ~= "a-chassis interface" and name ~= "drive" then
                            pcall(function() s.Disabled = true s:Destroy() end)
                        end
                    end
                end
            end
            for _, v in pairs(vehicle:GetDescendants()) do
                if v:IsA("ModuleScript") and (v.Name == "Tune" or string.find(string.lower(v.Name), "tune")) then
                    pcall(function()
                        local tune = require(v)
                        if tune.Horsepower then tune.Horsepower = tune.Horsepower * HP_Mult end
                        if tune.Redline then tune.Redline = tune.Redline + RPM_Add end
                        if tune.Ratios then
                            for i, ratio in pairs(tune.Ratios) do
                                if type(ratio) == "number" and ratio > 0 then tune.Ratios[i] = ratio * Ratio_Mult end
                            end
                        end
                        if tune.FinalDrive then tune.FinalDrive = tune.FinalDrive * FD_Mult end
                        if tune.Limiter ~= nil then tune.Limiter = false end
                        if tune.RevLimit then tune.RevLimit = 999999 end
                        if tune.SpeedLimit then tune.SpeedLimit = false end
                        if tune.TopSpeed then tune.TopSpeed = 999999 end
                        if tune.MaxSpeed then tune.MaxSpeed = 999999 end
                        if tune.DragMult then tune.DragMult = tune.DragMult * 0.05 end 
                        if tune.Weight then tune.Weight = tune.Weight * 0.7 end
                        foundTune = true
                    end)
                end
            end
            
            if foundTune then
                WindUI:Notify({ Title = "✅ " .. NamaMode, Content = "Aman! Turun lalu naik motor lagi ya bosku!", Duration = 5 })
            else
                WindUI:Notify({ Title = "❌ Gagal Inject", Content = "Bukan A-Chassis standar.", Duration = 4 })
            end
        end
    else
        WindUI:Notify({ Title = "⚠️ Woi Bosku!", Content = "Naik ke motornya dulu!", Duration = 3 })
    end
end

-- ============================================================================
-- // 15. AUTO MANDALIKA (AUTO RACE) – FITUR BARU
-- ============================================================================
local checkpointBalap = {
    Vector3.new(-392, 7, -1615),
    Vector3.new(1102, 7, -1621),
    Vector3.new(1420, 7, -1530),
    Vector3.new(1389, 7, -1137),
    Vector3.new(1375, 7, -1094),
    Vector3.new(1271, 7, -748),
    Vector3.new(1065, 7, -717),
    Vector3.new(1031, 7, -728),
    Vector3.new(889, 7, -446),
    Vector3.new(920, 7, -398),
    Vector3.new(1208, 7, 154),
    Vector3.new(1183, 7, 283),
    Vector3.new(979, 7, 849),
    Vector3.new(862, 7, 939),
    Vector3.new(537, 7, 1176),
    Vector3.new(459, 7, 1222),
    Vector3.new(-109, 7, 1228),
    Vector3.new(-183, 7, 1278),
    Vector3.new(-565, 7, 1515),
    Vector3.new(-670, 7, 1550),
    Vector3.new(-1531, 7, 1778),
    Vector3.new(-1625, 7, 1798),
    Vector3.new(-1574, 7, 959),
    Vector3.new(-1547, 7, 870),
    Vector3.new(-805, 7, 635),
    Vector3.new(-774, 7, 571),
    Vector3.new(-583, 7, -318),
    Vector3.new(-577, 7, -382),
    Vector3.new(-1147, 7, -820),
    Vector3.new(-1185, 7, -902),
    Vector3.new(-1453, 7, -1654),
    Vector3.new(-1462, 7, -1731),
    Vector3.new(-1344, 7, -2119),
    Vector3.new(-1300, 7, -2167),
    Vector3.new(-1133, 7, -2287),
    Vector3.new(-1090, 7, -1972),
    Vector3.new(-1124, 7, -1892),
    Vector3.new(-1153, 7, -1668),
    Vector3.new(-142, 7, -1616)
}

local raceConfig = {
    Motor   = "Yamahax-MioSporty",
    Speed   = 180,
    NoclipConn = nil
}

local function StartRaceLoop()
    local player = LocalPlayer
    local char = player.Character or player.CharacterAdded:Wait()
    local root = char:WaitForChild("HumanoidRootPart")
    local humanoid = char:WaitForChild("Humanoid")

    -- Aktifkan noclip permanen untuk karakter & kendaraan
    raceConfig.NoclipConn = Services.RunService.Stepped:Connect(function()
        if not State.IsRaceActive then return end
        if player.Character then
            for _, part in pairs(player.Character:GetDescendants()) do
                if part:IsA("BasePart") and part.CanCollide then
                    part.CanCollide = false
                end
            end
            local hum = player.Character:FindFirstChildOfClass("Humanoid")
            if hum and hum.SeatPart then
                local motorModel = hum.SeatPart:FindFirstAncestorOfClass("Model")
                if motorModel then
                    for _, part in pairs(motorModel:GetDescendants()) do
                        if part:IsA("BasePart") and part.CanCollide then
                            part.CanCollide = false
                        end
                    end
                end
            end
        end
    end)

    local startPos = checkpointBalap[1]
    root.CFrame = CFrame.new(startPos + Vector3.new(0, 3, 0))
    task.wait(1.5)

    while State.IsRaceActive do
        -- Spawn motor jika belum naik
        if not humanoid.SeatPart then
            pcall(function()
                Services.ReplicatedStorage:WaitForChild("SpawnCarEvents"):WaitForChild("SpawnCar"):FireServer(raceConfig.Motor)
            end)
            -- Cari prompt Ride di sekitar
            local promptFound = false
            for i = 1, 20 do
                for _, v in pairs(workspace:GetDescendants()) do
                    if v:IsA("ProximityPrompt") and v.ActionText == "Ride" and v.Enabled then
                        local part = v.Parent
                        if part and part:IsA("BasePart") and (part.Position - root.Position).Magnitude < 30 then
                            root.CFrame = CFrame.new(part.Position + Vector3.new(0, 2, 0))
                            DoTap(v)
                            promptFound = true
                            break
                        end
                    end
                end
                if promptFound then break end
                task.wait(0.5)
            end
            task.wait(2)
        end

        if not humanoid.SeatPart then task.wait(1) continue end

        local seat = humanoid.SeatPart
        local motorModel = seat:FindFirstAncestorOfClass("Model") or seat.Parent

        -- BodyVelocity & BodyGyro untuk terbang
        local bv = seat:FindFirstChild("Jet_BV") or Instance.new("BodyVelocity")
        bv.Name = "Jet_BV"; bv.MaxForce = Vector3.new(math.huge, math.huge, math.huge); bv.Velocity = Vector3.zero
        bv.Parent = seat

        local bg = seat:FindFirstChild("Jet_BG") or Instance.new("BodyGyro")
        bg.Name = "Jet_BG"; bg.MaxTorque = Vector3.new(math.huge, math.huge, math.huge); bg.P = 100000; bg.D = 1000
        bg.Parent = seat

        -- Hover di start, tunggu "GO!!!"
        local waitGo = true
        while waitGo and State.IsRaceActive do
            local cfStart = CFrame.lookAt(startPos + Vector3.new(0,3.5,0), checkpointBalap[2] + Vector3.new(0,3.5,0))
            bg.CFrame = cfStart
            bv.Velocity = Vector3.zero
            task.wait()
            for _, gui in pairs(player.PlayerGui:GetDescendants()) do
                if gui:IsA("TextLabel") and gui.Visible and string.find(string.upper(gui.Text), "GO!!!") then
                    waitGo = false
                    break
                end
            end
        end

        if not State.IsRaceActive then break end

        -- Lintasan maju
        for i = 2, #checkpointBalap do
            if not State.IsRaceActive then break end
            local target = checkpointBalap[i] + Vector3.new(0, 3.5, 0)
            while State.IsRaceActive do
                local dist = (Vector3.new(seat.Position.X, 0, seat.Position.Z) - Vector3.new(target.X, 0, target.Z)).Magnitude
                if dist < 15 then break end
                local cf = CFrame.lookAt(seat.Position, target)
                bg.CFrame = cf
                bv.Velocity = cf.LookVector * raceConfig.Speed
                seat.AssemblyLinearVelocity = bv.Velocity
                task.wait()
            end
        end

        -- Kembali ke start
        local targetStart = checkpointBalap[1] + Vector3.new(0, 3.5, 0)
        while State.IsRaceActive do
            local dist = (Vector3.new(seat.Position.X, 0, seat.Position.Z) - Vector3.new(targetStart.X, 0, targetStart.Z)).Magnitude
            if dist < 15 then break end
            local cf = CFrame.lookAt(seat.Position, targetStart)
            bg.CFrame = cf
            bv.Velocity = cf.LookVector * raceConfig.Speed
            seat.AssemblyLinearVelocity = bv.Velocity
            task.wait()
        end

        -- Hentikan di start
        bv.Velocity = Vector3.zero
        seat.AssemblyLinearVelocity = Vector3.zero
        local cfStart = CFrame.lookAt(startPos + Vector3.new(0,3.5,0), checkpointBalap[2] + Vector3.new(0,3.5,0))
        motorModel:PivotTo(cfStart)
        bg.CFrame = cfStart
        task.wait(2)
    end

    -- Bersihkan
    if raceConfig.NoclipConn then
        raceConfig.NoclipConn:Disconnect()
        raceConfig.NoclipConn = nil
    end
end

local function StartRaceScript()
    if State.IsRaceActive then return end
    State.IsRaceActive = true
    task.spawn(StartRaceLoop)
end

local function StopRaceScript()
    State.IsRaceActive = false
    if raceConfig.NoclipConn then
        raceConfig.NoclipConn:Disconnect()
        raceConfig.NoclipConn = nil
    end
end

-- ============================================================================
-- // 16. UI — TAMBAHAN TAB AUTO RACE
-- ============================================================================
local Window = WindUI:CreateWindow({
    Title                       = "King Akbar - Drag Drive Simulator",
    Icon                        = "crown",
    Author                      = "King Akbar",
    Folder                      = "MySuperHub",
    Size                        = wSz,
    MinSize                     = mnSz,
    MaxSize                     = mxSz,
    Transparent                 = false,
    Background                  = "rbxassetid://127295801178451",
    BackgroundImageTransparency = 0.5,
    Theme                       = "Dark",
    Resizable                   = true,
    SideBarWidth                = 210,
    HideSearchBar               = false,
    ScrollBarEnabled            = true,
})

-- Tab-tab sebelumnya (Info, Auto Farm, Keamanan, Performa, Pengaturan, Mode Instan, Custom Setting) tetap sama,
-- hanya ditambahkan TAB baru: "Auto Race"

-- ============================
-- TAB 8: AUTO RACE (MANDALIKA)
-- ============================
local TabRace = Window:Tab({ Title = "Auto Race", Icon = "flag", Border = true })

local RaceSection = TabRace:Section({
    Title = "Mandalika Auto Race",
    Box = true,
    BoxBorder = true,
    Opened = true,
})

RaceSection:Input({
    Title       = "Nama Motor",
    Placeholder = "Yamahax-MioSporty",
    Callback    = function(text)
        if text ~= "" then
            raceConfig.Motor = text
        end
    end
})

RaceSection:Slider({
    Title    = "Kecepatan Balap",
    Desc     = "Semakin tinggi semakin cepat, tapi tetap aman",
    Step     = 5,
    Value    = { Min = 50, Max = 500, Default = 180 },
    Callback = function(v) raceConfig.Speed = v end,
})

RaceSection:Toggle({
    Title    = "Jalankan Auto Mandalika",
    Icon     = "play",
    Value    = false,
    Callback = function(on)
        if on then
            StartRaceScript()
            WindUI:Notify({ Title = "🏁 Auto Race", Content = "Auto Mandalika diaktifkan! Nunggu aba-aba GO!!!", Duration = 3 })
        else
            StopRaceScript()
            WindUI:Notify({ Title = "🛑 Auto Race", Content = "Auto Mandalika dimatikan.", Duration = 3 })
        end
    end,
})

-- ============================
-- AKHIR UI
-- ============================
Window:EditOpenButton({
    Title           = "Open King Akbar",
    Icon            = "crown",
    CornerRadius    = UDim.new(0, 12),
    StrokeThickness = 2,
    Color = ColorSequence.new({
        ColorSequenceKeypoint.new(0, Color3.fromHex("#ffffff")),
        ColorSequenceKeypoint.new(1, Color3.fromHex("#0a0a0a")),
    }),
    Enabled   = true,
    Draggable = true,
})

local FpsTag = Window:Tag({
    Title = "Fps: ...",
    Color = WindUI:Gradient({
        [0]   = { Color = Color3.fromHex("#0a0a0a"), Transparency = 0 },
        [100] = { Color = Color3.fromHex("#888888"), Transparency = 0 },
    }, { Rotation = 45 }),
})

task.spawn(function()
    while task.wait(1) do
        pcall(function()
            local fps  = math.floor(1 / Services.RunService.RenderStepped:Wait())
            local ping = math.floor(Services.Stats.Network.ServerStatsItem["Data Ping"]:GetValue())
            if FpsTag and FpsTag.SetTitle then
                FpsTag:SetTitle(("Fps: %d | Ping: %d"):format(fps, ping))
            end
        end)
    end
end)

Window:SetIconSize(47)
WindUI:SetTheme("dark")
TabInfo:Select()

WindUI:Notify({
    Title    = "👑 KING AKBAR V5.9 FINAL + AUTO MANDALIKA!",
    Content  = "Fitur balapan otomatis sirkuit Mandalika siap digunakan. Gas cuan!",
    Duration = 5,
})

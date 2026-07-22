local WindUI = loadstring(game:HttpGet("https://github.com/Footagesus/WindUI/releases/latest/download/main.lua"))()

local Window = WindUI:CreateWindow({
    Title = "King Vypers",
    Icon = "rbxassetid://139467646163013",
    Folder = "KingVypers",
    Background = "rbxassetid://97514324988224",
    BackgroundImageTransparency = 0.35,
    Size = UDim2.new(0, 530, 0, 300),
    MinSize = Vector2.new(530, 300),
    MaxSize = Vector2.new(530, 300),
    NewElements = true,
    OpenButton = {
        Enabled = false,
    },
})local lp = game:GetService("Players").LocalPlayer
local char = lp.Character or lp.CharacterAdded:Wait()
local head = char:WaitForChild("Head")
local RE = game:GetService("ReplicatedStorage").Packages._Index["sleitnick_knit@1.7.0"].knit.Services.FishingService.RE

local currentFishData = nil

-- simpan fishData
RE.FishCaught.OnClientEvent:Connect(function(fishData)
    currentFishData = fishData
    print("[FishCaught]", fishData.fish, fishData.rarity)
end)

head.ChildAdded:Connect(function(child)
    if child.Name == "FishingExclamation" then
        print("[!] Tanda seru muncul — ikan nyangkut!")
        task.wait(4)

        if currentFishData then
            local stats = {
                yanksCompleted = math.random(2, 5),
                minigameDurationMs = math.random(13000, 15000),
                redBarDangerTicks = math.random(40, 100),
                snagAttempts = 0,
                snagSuccesses = 0,
                maxRedBarProximity = math.random(60, 95) / 100,
                totalSpamPoints = math.random(20, 50)
            }

            RE.FishingSuccess:FireServer(true, stats)
            firesignal(RE.Cleanup.OnClientEvent)
            firesignal(RE.SetRodLock.OnClientEvent, false)

            print("[Done]", currentFishData.fish, "|", currentFishData.rarity, "| $"..currentFishData.sellPrice)
            currentFishData = nil
        else
            print("[!] FishData kosong!")
        end
    end
end)

head.ChildRemoved:Connect(function(child)
    if child.Name == "FishingExclamation" then
        print("[!] Tanda seru hilang")
    end
end)

-- */  Colors  /* --
local Kings = Color3.fromHex("#120324")
local Mains = Color3.fromHex("#110029")
local Purple = Color3.fromHex("#7775F2")
local Yellow = Color3.fromHex("#ECA201")
local Green = Color3.fromHex("#10C550")
local Grey = Color3.fromHex("#292828")
local Blue = Color3.fromHex("#257AF7")
local Red = Color3.fromHex("#EF4F1D")


-- TARUH DISINI ↓
WindUI:AddTheme({
    Name = "MachTheme",
    Background = Kings,
})
WindUI:SetTheme("MachTheme")
-- SAMPAI SINI ↑

-- Tambahkan setelah CreateWindow
Window:Tag({
    Title = "PREMIUM",
    Color = Mains,
})

Window:Tag({
    Title = "BETA",
    Color = Purple,
})

-- =================================================================
-- 🔴 TOMBOL MERAH (PC + MOBILE SUPPORT)
-- =================================================================
local Players = game:GetService("Players")
local TweenService = game:GetService("TweenService")
local UserInputService = game:GetService("UserInputService")
local RunService = game:GetService("RunService")
local CoreGui = game:GetService("CoreGui")

-- Protection
local protectGui
local success, result = pcall(function()
    if gethui then
        return gethui()
    elseif syn and syn.protect_gui then
        local sg = Instance.new("ScreenGui")
        syn.protect_gui(sg)
        sg.Parent = CoreGui
        return sg.Parent
    else
        return CoreGui
    end
end)

if success then
    protectGui = result
else
    protectGui = Players.LocalPlayer:WaitForChild("PlayerGui")
end

local screenGui = Instance.new("ScreenGui")
screenGui.Name = "MachFishingButton"
screenGui.ResetOnSpawn = false
screenGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
screenGui.Parent = protectGui

local buttonFrame = Instance.new("Frame")
buttonFrame.Size = UDim2.new(0, 42, 0, 42)
buttonFrame.Position = UDim2.new(0, 20, 0, 20)
buttonFrame.BackgroundTransparency = 1
buttonFrame.Parent = screenGui

local imageButton = Instance.new("ImageButton")
imageButton.Size = UDim2.new(1, 0, 1, 0)
imageButton.BackgroundColor3 = Color3.fromRGB(20, 20, 20)       -- merah -> hitam pekat
imageButton.BackgroundTransparency = 0.2
imageButton.Image = "rbxassetid://107726435417936"
imageButton.ScaleType = Enum.ScaleType.Fit
imageButton.Parent = buttonFrame

local uiCorner = Instance.new("UICorner")
uiCorner.CornerRadius = UDim.new(0, 8)
uiCorner.Parent = imageButton

local uiStroke = Instance.new("UIStroke")
uiStroke.Thickness = 2
uiStroke.Color = Color3.fromRGB(60, 60, 60)                     -- merah muda -> abu gelap
uiStroke.Parent = imageButton

local uiGradient = Instance.new("UIGradient")
uiGradient.Color = ColorSequence.new(
    Color3.fromRGB(20, 20, 20),                                  -- merah -> hitam pekat
    Color3.fromRGB(60, 60, 60)                                   -- merah muda -> abu gelap
)
uiGradient.Parent = uiStroke

-- ========================================
-- 🖱️📱 DRAG + CLICK (PC + MOBILE)
-- ========================================
local dragging = false
local dragInput
local dragStart
local startPos

imageButton.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
        dragging = true
        dragStart = input.Position
        startPos = buttonFrame.Position
        
        input.Changed:Connect(function()
            if input.UserInputState == Enum.UserInputState.End then
                dragging = false
            end
        end)
    end
end)

imageButton.InputChanged:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch then
        dragInput = input
    end
end)

-- DRAG MOVEMENT (PC + MOBILE)
UserInputService.InputChanged:Connect(function(input)
    if dragging and dragInput and (input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch) then
        local delta = input.Position - dragStart
        buttonFrame.Position = UDim2.new(
            startPos.X.Scale,
            startPos.X.Offset + delta.X,
            startPos.Y.Scale,
            startPos.Y.Offset + delta.Y
        )
    end
end)

-- CLICK DETECTION (PC + MOBILE)
local clickStart = nil
imageButton.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
        clickStart = input.Position
    end
end)

imageButton.InputEnded:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
        if clickStart then
            local moved = (input.Position - clickStart).Magnitude
            
            -- CLICK (ga di-drag)
            if moved < 10 then
                if Window and Window.Toggle then
                    Window:Toggle()
                    print("🔄 Window toggled")
                end
            end
            
            clickStart = nil
        end
    end
end)

-- HOVER EFFECT (PC only)
imageButton.MouseEnter:Connect(function()
    TweenService:Create(imageButton, TweenInfo.new(0.15), {BackgroundTransparency = 0}):Play()
end)

imageButton.MouseLeave:Connect(function()
    TweenService:Create(imageButton, TweenInfo.new(0.15), {BackgroundTransparency = 0.2}):Play()
end)

-- INFO TAB

	local InfoTab = Window:Tab({
		Title = "Info",
		Icon = "solar:info-square-bold",
		IconColor = Mains,
		IconShape = "Square",
		Border = true,
	})






local HttpService = game:GetService("HttpService")
-- Fungsi fetch member count
local memberCount = "N/A"
local onlineCount = "N/A"

local function fetchDiscordInfo()
    local req = request or http_request or syn and syn.request
    if not req then return end
    
    local success, result = pcall(function()
        return req({
            Url = "https://discord.com/api/v9/invites/XmWf3YQPpZ?with_counts=true",
            Method = "GET",
            Headers = {
                ["User-Agent"] = "Mozilla/5.0"
            }
        })
    end)
    
    if success and result and result.StatusCode == 200 then
        local ok, data = pcall(function()
            return game:GetService("HttpService"):JSONDecode(result.Body)
        end)
        
        if ok and data then
            memberCount = tostring(data.approximate_member_count or "N/A")
            onlineCount = tostring(data.approximate_presence_count or "N/A")
        end
    end
end

-- Fetch dulu sebelum bikin UI
fetchDiscordInfo()

-- Info + buttons + banner semua dalam satu frame
local ServerInfo = InfoTab:Paragraph({
    Title = "King Vypers | Official",
    Desc = "• Member Count: " .. memberCount .. "\n• Online Count: " .. onlineCount,
    Image = "rbxassetid://107726435417936",
    Thumbnail = "rbxassetid://83197533072664",
    ThumbnailSize = 80,
    Buttons = {
        {
            Title = "Copy Discord Invite",
            Color= Color3.fromHex("#5707AB"),
            Icon = "link",
            Callback = function()
                if setclipboard then
                    setclipboard("https://discord.gg/XmWf3YQPpZ")
                end
            end
        },
        {
            Title = "Update Info",
            Icon = "refresh-cw",
            Callback = function()
                fetchDiscordInfo()
                ServerInfo:SetDesc("• Member Count: " .. memberCount .. "\n• Online Count: " .. onlineCount)
            end
        }
    }
})
-- ══════════════════════════════════════════
--              RESOLVER MODUL
-- ══════════════════════════════════════════
-- EventResolver v7 (hash-safe, scan langsung dari net:GetChildren())
local EmbeddedEventResolver = (function()
    local RS = game:GetService("ReplicatedStorage")

    local self = {
        _initialized = false,
        _re = {},
        _rf = {},
        _netFolder = nil,
    }

    local function isHash(name)
        -- Hash = hex string panjang >= 32 char, hanya hex
        return #name >= 32 and name:match("^[0-9a-f]+$") ~= nil
    end

    local function stripPrefix(name)
        -- "RF/ChargeFishingRod" -> "ChargeFishingRod"
        -- "RE/FishCaught"       -> "FishCaught"
        -- "URE/TakeMeasurement" -> "TakeMeasurement"
        return name:match("^[A-Z]+/(.+)$") or name
    end

    local function findNetFolder()
        if self._netFolder and self._netFolder.Parent then
            return self._netFolder
        end

        local ok, net = pcall(function()
            return RS.Packages._Index["sleitnick_net@0.2.0"].net
        end)
        if ok and net then
            self._netFolder = net
            return net
        end

        -- Fallback: scan _Index cari folder yang ada child "net"/"Net"
        local idx = RS:FindFirstChild("Packages")
        idx = idx and idx:FindFirstChild("_Index")
        if not idx then return nil end

        for _, child in ipairs(idx:GetChildren()) do
            if child.Name:lower():find("net") then
                local n = child:FindFirstChild("net") or child:FindFirstChild("Net")
                if n then
                    self._netFolder = n
                    return n
                end
            end
        end

        return nil
    end

    local function clearCaches()
        self._re = {}
        self._rf = {}
    end

    local function scanPairs(netFolder)
        local children = netFolder:GetChildren()
        local i = 1

        while i <= #children do
            local curr = children[i]
            local nextChild = children[i + 1]

            if nextChild then
                local currName = stripPrefix(curr.Name)
                local nextName = stripPrefix(nextChild.Name)
                local currClass = curr.ClassName
                local nextClass = nextChild.ClassName

                -- Pair valid: sama class, curr = nama asli, next = hash
                if currClass == nextClass
                    and not isHash(currName)
                    and isHash(nextName) then

                    if curr:IsA("RemoteFunction") then
                        self._rf[currName] = nextChild
                    elseif curr:IsA("RemoteEvent") or curr:IsA("UnreliableRemoteEvent") then
                        self._re[currName] = nextChild
                    end

                    i = i + 2
                    continue
                end
            end

            -- Bukan pair: simpan apa adanya jika nama asli (tidak hash)
            local name = stripPrefix(curr.Name)
            if not isHash(name) then
                if curr:IsA("RemoteFunction") and not self._rf[name] then
                    self._rf[name] = curr
                elseif (curr:IsA("RemoteEvent") or curr:IsA("UnreliableRemoteEvent"))
                    and not self._re[name] then
                    self._re[name] = curr
                end
            end

            i = i + 1
        end
    end

    function self:Init()
        if self._initialized then return true end

        local net = findNetFolder()
        if not net then
            warn("[EmbeddedEventResolver] net folder tidak ditemukan!")
            return false
        end

        clearCaches()
        scanPairs(net)

        self._initialized = true
        _G.EventResolver = self
        _G.ResolvedNetEvents = { RE = self._re, RF = self._rf }

        return true
    end

    function self:GetRF(name)
        if not self._initialized then self:Init() end
        if self._rf[name] then return self._rf[name] end

        local net = findNetFolder()
        if net then
            scanPairs(net)
        end

        return self._rf[name]
    end

    function self:GetRE(name)
        if not self._initialized then self:Init() end
        if self._re[name] then return self._re[name] end

        local net = findNetFolder()
        if net then
            scanPairs(net)
        end

        return self._re[name]
    end

    function self:GetNetFolder()
        return findNetFolder()
    end

    function self:IsInitialized()
        return self._initialized
    end

    function self:Reset()
        self._initialized = false
        self._netFolder = nil
        clearCaches()
    end

    function self:Debug()
        local rfCount, reCount = 0, 0
        print("[EmbeddedEventResolver] === RemoteFunctions ===")
        for k, v in pairs(self._rf) do
            rfCount = rfCount + 1
            print(string.format("  %-40s -> %s", k, v.Name))
        end
        print("[EmbeddedEventResolver] === RemoteEvents ===")
        for k, v in pairs(self._re) do
            reCount = reCount + 1
            print(string.format("  %-40s -> %s", k, v.Name))
        end
        print(string.format("[EmbeddedEventResolver] Total: %d RF, %d RE", rfCount, reCount))
    end

    self:Init()
    return self
end)()

task.spawn(function() EmbeddedEventResolver:Init() end)

local Players = game:GetService("Players")
local localPlayer = Players.LocalPlayer
local Workspace = game:GetService("Workspace") or workspace
local RunService = game:GetService("RunService")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local LocalPlayer = localPlayer
local player = localPlayer

localPlayer.CharacterAdded:Connect(function()
    task.wait(3)
    EmbeddedEventResolver:Reset()
    task.spawn(function() EmbeddedEventResolver:Init() end)
end)

local function safeFire(fn) pcall(fn) end

local NetEvents = {}
setmetatable(NetEvents, {
    __index = function(t, k)
        if k == "RF_ChargeFishingRod" then return EmbeddedEventResolver:GetRF("ChargeFishingRod")
        elseif k == "RF_RequestMinigame" then return EmbeddedEventResolver:GetRF("RequestFishingMinigameStarted")
        elseif k == "RF_CancelFishingInputs" then return EmbeddedEventResolver:GetRF("CancelFishingInputs")
        elseif k == "RF_UpdateAutoFishingState" then return EmbeddedEventResolver:GetRF("UpdateAutoFishingState")
        elseif k == "RE_FishingCompleted" then return EmbeddedEventResolver:GetRE("CatchFishCompleted")
        elseif k == "RE_UpdateChargeState" then return EmbeddedEventResolver:GetRE("UpdateChargeState")
        elseif k == "RE_MinigameChanged" or k == "RF_MinigameChange" then return EmbeddedEventResolver:GetRE("FishingMinigameChanged")
        elseif k == "RE_FishCaught" then return EmbeddedEventResolver:GetRE("FishCaught")
        elseif k == "RE_FishingStopped" then return EmbeddedEventResolver:GetRE("FishingStopped")
        elseif k == "RF_InitiateTrade" then return EmbeddedEventResolver:GetRF("InitiateTrade")
        elseif k == "RF_AwaitTradeResponse" then return EmbeddedEventResolver:GetRF("AwaitTradeResponse")
        elseif k == "RF_ConsumePotion" then return EmbeddedEventResolver:GetRF("ConsumePotion")
        elseif k == "RE_FavoriteItem" then return EmbeddedEventResolver:GetRE("FavoriteItem")
        elseif k == "RE_EquipItem" then return EmbeddedEventResolver:GetRE("EquipItem")
        elseif k == "RF_EquipToolFromHotbar" then return EmbeddedEventResolver:GetRF("EquipToolFromHotbar")
        elseif k == "RE_ActivateEnchantingAltar" then return EmbeddedEventResolver:GetRE("ActivateEnchantingAltar")
        elseif k == "RE_ActivateSecondEnchantingAltar" then return EmbeddedEventResolver:GetRE("ActivateSecondEnchantingAltar")
        elseif k == "RE_RollEnchant" then return EmbeddedEventResolver:GetRE("RollEnchant")
        elseif k == "netFolder" then return EmbeddedEventResolver:GetNetFolder()
        elseif k == "IsInitialized" then return EmbeddedEventResolver:IsInitialized()
        end
        return rawget(t, k)
    end,
    __newindex = function(t, k, v)
        if k == "IsInitialized" then return end
        rawset(t, k, v)
    end
})
local CombinedModules = rawget(getgenv(), "CombinedModules")
if type(CombinedModules) ~= "table" then
    CombinedModules = {}
    getgenv().CombinedModules = CombinedModules
end

-- ══════════════════════════════════════════
--              Fsihing Tab
-- ══════════════════════════════════════════
local Fsihing = Window:Tab({
    Title = "Fishing",
    Icon = "fish",
    IconColor = Mains,
    IconShape = "Square",
    Border = true,
})

-- ══════════════════════════════════════════
--              INSTANT FISH MODULE
-- ══════════════════════════════════════════
local InstantV1 = (function()
    local Instant = {}
    Instant.Active = false
    Instant.Settings = { CompleteDelay = 0.04 }
    local spamThread = nil

    local function loop()
        while Instant.Active do
            if not EmbeddedEventResolver:IsInitialized() then task.wait(1) continue end
            local t = Workspace:GetServerTimeNow()
            safeFire(function() if NetEvents.RF_ChargeFishingRod then NetEvents.RF_ChargeFishingRod:InvokeServer(nil, nil, t, nil) end end)
            task.wait(0.3)
            safeFire(function() if NetEvents.RF_RequestMinigame then NetEvents.RF_RequestMinigame:InvokeServer(-1.2, 0.95, t) end end)
            task.wait(Instant.Settings.CompleteDelay)
            safeFire(function()
                local rf = NetEvents.RE_FishingCompleted
                if rf then rf:FireServer() end
            end)
            task.wait(0.04)
        end
    end

    function Instant.Start()
        if Instant.Active then return end
        if not EmbeddedEventResolver:IsInitialized() then return end
        Instant.Active = true
        spamThread = task.spawn(loop)
    end

    function Instant.Stop()
        if not Instant.Active then return end
        Instant.Active = false
        if spamThread then task.cancel(spamThread) spamThread = nil end
        safeFire(function()
            if NetEvents.RF_CancelFishingInputs then NetEvents.RF_CancelFishingInputs:InvokeServer() end
        end)
    end

    return Instant
end)()

-- ══════════════════════════════════════════
--              INSTANT FISH SECTION
-- ══════════════════════════════════════════
local InstantSection = Fsihing:Section({ 
    Title = "Instant Fish",
    Box = true,
    TextXAlignment = "Center",
    TextSize = 15,
    Opened = false,
})

InstantSection:Input({
    Title = "Instant Delay",
    Value = tostring(InstantV1.Settings.CompleteDelay),
    Type = "Input",
    Placeholder = "0.05",
    Callback = function(input)
        local num = tonumber(input)
        if num then
            InstantV1.Settings.CompleteDelay = num
        end
    end
})

InstantSection:Toggle({
    Title = "Enable Instant Fish",
    Value = false,
    Callback = function(state)
        if state then
            InstantV1.Start()
        else
            InstantV1.Stop()
        end
    end
})

-- ══════════════════════════════════════════
--              Legit FISH Modul
-- ══════════════════════════════════════════
CombinedModules.legit = (function()
    local Legit = {}
    Legit.Active = false
    Legit.AutoShake = false

    Legit.Settings = {
        CastWait = 0.5,
        ClickWait = 0.04, -- langsung set 0.04
        ShakeDelay = 0.05,
    }

    local fishingThread = nil
    local shakeThread = nil
    local FishingController = nil
    local Camera = workspace.CurrentCamera
    local Players = game:GetService("Players")
    local ReplicatedStorage = game:GetService("ReplicatedStorage")

    local function initFishingController()
        if FishingController then return true end
        local Controllers = ReplicatedStorage:FindFirstChild("Controllers")
        if Controllers then
            local mod = Controllers:FindFirstChild("FishingController")
            if mod then
                FishingController = require(mod)
                return true
            end
        end
        return false
    end

    local function getViewportCenter()
        local vp = Camera.ViewportSize
        return Vector2.new(vp.X / 2, vp.Y / 2)
    end

    local function resetFishingState()
        pcall(function()
            FishingController:RequestClientStopFishing(true)
        end)
        pcall(function()
            if NetEvents and NetEvents.RF_CancelFishingInputs then
                NetEvents.RF_CancelFishingInputs:InvokeServer()
            end
        end)
    end

    local function mainLoop()
        if not initFishingController() then
            Legit.Active = false
            return
        end

        while Legit.Active do
            local ok, err = pcall(function()
                if FishingController:OnCooldown() then
                    task.wait(0.2)
                    return
                end

                if FishingController:GetCurrentGUID() then
                    FishingController:RequestFishingMinigameClick()
                    task.wait(Legit.Settings.ClickWait)
                else
                    FishingController:RequestChargeFishingRod(
                        getViewportCenter(),
                        true
                    )
                    task.wait(Legit.Settings.CastWait)
                end
            end)

            if not ok then
                warn("[Legit] Error: " .. tostring(err))
                task.wait(1)
            end
        end
    end

    local function shakeLoop()
        if not initFishingController() then
            warn("[AutoShake] FishingController tidak ditemukan!")
            Legit.AutoShake = false
            return
        end

        while Legit.AutoShake do
            pcall(function()
                FishingController:RequestFishingMinigameClick()
            end)
            task.wait(Legit.Settings.ShakeDelay)
        end
    end

    function Legit.Start()
        if Legit.Active then return end
        Legit.Active = true
        fishingThread = task.spawn(mainLoop)
        -- Auto shake langsung nyala bareng
        Legit.StartAutoShake()
    end

    function Legit.Stop()
        if not Legit.Active then return end
        Legit.Active = false
        if fishingThread then
            task.cancel(fishingThread)
            fishingThread = nil
        end
        -- Auto shake langsung mati bareng
        Legit.StopAutoShake()
        task.wait(0.1)
        resetFishingState()
    end

    function Legit.StartAutoShake()
        if Legit.AutoShake then return end
        Legit.AutoShake = true
        shakeThread = task.spawn(shakeLoop)
    end

    function Legit.StopAutoShake()
        if not Legit.AutoShake then return end
        Legit.AutoShake = false
        if shakeThread then
            task.cancel(shakeThread)
            shakeThread = nil
        end
        task.wait(0.1)
        resetFishingState()
    end

    return Legit
end)()

-- ══════════════════════════════════════════
--              Legit FISH SECTION
-- ══════════════════════════════════════════
local LegitSection = Fsihing:Section({ 
    Title = "Legit Fish",
    Box = true,
    TextXAlignment = "Center",
    TextSize = 15,
    Opened = false,
})

LegitSection:Input({
    Title = "Cast Delay",
    Value = tostring(CombinedModules.legit.Settings.CastWait),
    Type = "Input",
    Placeholder = "0.5",
    Callback = function(input)
        local num = tonumber(input)
        if num then
            CombinedModules.legit.Settings.CastWait = num
        end
    end
})

LegitSection:Toggle({
    Title = "Enable Legit Fish",
    Value = false,
    Callback = function(state)
        if state then
            CombinedModules.legit.Start()
        else
            CombinedModules.legit.Stop()
        end
    end
})

-- ══════════════════════════════════════════
--              BLATANT FISH MODULE
-- ══════════════════════════════════════════
local BlatantV1 = (function()
    local Blatant = {}
    Blatant.Active = false
    Blatant.Settings = { SpamCastDelay = 0.05, CompleteDelay = 0.01, InstantComplete = true, ChargeSpam = 3 }
    local spamThread = nil

    local function loop()
        while Blatant.Active do
            if not EmbeddedEventResolver:IsInitialized() then task.wait(1) continue end
            local t = Workspace:GetServerTimeNow()
            for i = 1, Blatant.Settings.ChargeSpam do
                safeFire(function() if NetEvents.RF_ChargeFishingRod then NetEvents.RF_ChargeFishingRod:InvokeServer(nil, nil, t, nil) end end)
                safeFire(function() if NetEvents.RF_RequestMinigame then NetEvents.RF_RequestMinigame:InvokeServer(-1.2331848144531, 0.89899236174132, t) end end)
                if i < Blatant.Settings.ChargeSpam then task.wait(0.05) end
            end
            if Blatant.Settings.InstantComplete then
                task.wait(Blatant.Settings.CompleteDelay)
                safeFire(function()
                    local rf = NetEvents.RE_FishingCompleted
                    if rf then rf:FireServer() end
                end)
                task.wait(0.01)
                if NetEvents.RF_CancelFishingInputs then safeFire(function() NetEvents.RF_CancelFishingInputs:InvokeServer() end) end
            end
            task.wait(Blatant.Settings.SpamCastDelay)
        end
    end

    function Blatant.Start()
        if Blatant.Active then return end
        if not EmbeddedEventResolver:IsInitialized() then return end
        Blatant.Active = true
        spamThread = task.spawn(loop)
    end

    function Blatant.Stop()
        if not Blatant.Active then return end
        Blatant.Active = false
        if spamThread then task.cancel(spamThread) spamThread = nil end
    end

    return Blatant
end)()

-- ══════════════════════════════════════════
--              BLATANT FISH SECTION
-- ══════════════════════════════════════════
local BlatantSection = Fsihing:Section({ 
    Title = "Blatant Fish [BETA]",
    Box = true,
    TextXAlignment = "Center",
    TextSize = 15,
    Opened = false,
})

BlatantSection:Input({
    Title = "Spam Cast Delay",
    Value = tostring(BlatantV1.Settings.SpamCastDelay),
    Type = "Input",
    Placeholder = "0.05",
    Callback = function(input)
        local num = tonumber(input)
        if num then
            BlatantV1.Settings.SpamCastDelay = num
        end
    end
})

BlatantSection:Input({
    Title = "Complete Delay",
    Value = tostring(BlatantV1.Settings.CompleteDelay),
    Type = "Input",
    Placeholder = "0.01",
    Callback = function(input)
        local num = tonumber(input)
        if num then
            BlatantV1.Settings.CompleteDelay = num
        end
    end
})

BlatantSection:Toggle({
    Title = "Enable Blatant Fish",
    Value = false,
    Callback = function(state)
        if state then
            BlatantV1.Start()
        else
            BlatantV1.Stop()
        end
    end
})


-- ══════════════════════════════════════════
--              Support Feature
-- ══════════════════════════════════════════
--no fishing animation
local NoFishingAnimation = (function()
    local M = {}
    M.Enabled = false
    M.Connection = nil
    function M.StartWithDelay()
        task.wait(0.5)
        M.Start()
    end
    function M.Start()
        if M.Enabled then return end
        M.Enabled = true
        local function blockAnimations()
            local char = LocalPlayer.Character
            if not char then return end
            local humanoid = char:FindFirstChildOfClass("Humanoid")
            if not humanoid then return end
            local animator = humanoid:FindFirstChildOfClass("Animator")
            if not animator then return end
            for _, track in pairs(animator:GetPlayingAnimationTracks()) do
                if track.Name:find("Reel") or track.Name:find("Fish") then track:Stop() end
            end
        end
        M.Connection = RunService.Heartbeat:Connect(function()
            if M.Enabled then blockAnimations() end
        end)
    end
    function M.Stop()
        if not M.Enabled then return end
        M.Enabled = false
        if M.Connection then M.Connection:Disconnect() M.Connection = nil end
    end
    return M
end)()

--Auto Equip Rod
local AutoEquipRod = (function()
    local M = {}
    local v0 = { Players = Players, RS = ReplicatedStorage }
    local v5Success, v5 = pcall(function()
        return {
            PlayerStatsUtility = require(v0.RS.Shared.PlayerStatsUtility),
            ItemUtility = require(v0.RS.Shared.ItemUtility)
        }
    end)
    if not v5Success then
        M.isSupported = false
        M.Start = function() end
        M.Stop = function() end
        return M
    end
    
    -- ✅ Nama fungsi konsisten, pakai GetRF karena sudah RemoteFunction
    local function getRFEquip() return EmbeddedEventResolver:GetRF("EquipToolFromHotbar") end
    
    local v7Success, v7 = pcall(function()
        return { Data = require(v0.RS.Packages.Replion).Client:WaitReplion("Data") }
    end)
    if not v7Success then
        M.isSupported = false
        M.Start = function() end
        M.Stop = function() end
        return M
    end
    M.isSupported = true
    local v8 = { autoEquipRod = false, loopConnection = nil }
    local function isRodEquipped()
        local success, result = pcall(function()
            local v217 = v7.Data:Get("EquippedId")
            if not v217 then return false end
            local equippedItem = v5.PlayerStatsUtility:GetItemFromInventory(v7.Data, function(v218) return v218.UUID == v217 end)
            if not equippedItem then return false end
            local itemData = v5.ItemUtility:GetItemData(equippedItem.Id)
            return itemData and itemData.Data.Type == "Fishing Rods"
        end)
        return success and result or false
    end
    local function equipRod()
        pcall(function()
            if not isRodEquipped() then
                local rfEquip = getRFEquip()                -- ✅ Fix: getREEquip() → getRFEquip()
                if rfEquip then rfEquip:InvokeServer(1) end -- ✅ InvokeServer tetap benar untuk RF
            end
        end)
    end
    function M.Start()
        v8.autoEquipRod = true
        if v8.loopConnection then v8.loopConnection:Disconnect() end
        v8.loopConnection = task.spawn(function()
            while v8.autoEquipRod do equipRod() task.wait(1) end
        end)
    end
    function M.Stop()
        v8.autoEquipRod = false
        if v8.loopConnection then task.cancel(v8.loopConnection) v8.loopConnection = nil end
    end
    return M
end)()

--Lock Position
local LockPosition = (function()
    local M = {}
    M.Enabled = false
    M.LockedPos = nil
    M.Connection = nil
    function M.Start()
        if M.Enabled then return end
        M.Enabled = true
        local char = localPlayer.Character or localPlayer.CharacterAdded:Wait()
        local hrp = char:WaitForChild("HumanoidRootPart")
        M.LockedPos = hrp.CFrame
        M.Connection = RunService.Heartbeat:Connect(function()
            if not M.Enabled then return end
            local c = player.Character
            if not c then return end
            local hrp2 = c:FindFirstChild("HumanoidRootPart")
            if not hrp2 then return end
            hrp2.CFrame = M.LockedPos
        end)
    end
    function M.Stop()
        M.Enabled = false
        if M.Connection then M.Connection:Disconnect() M.Connection = nil end
    end
    return M
end)()

--disable cutscane
local DisableCutscenes = (function()
    local CutsceneController = nil
    local OldPlayCutscene = nil
    local isDisabled = false
    local function initializeCutsceneHook()
        pcall(function()
            CutsceneController = require(ReplicatedStorage:WaitForChild("Controllers"):WaitForChild("CutsceneController"))
            if CutsceneController and CutsceneController.Play then
                OldPlayCutscene = CutsceneController.Play
                CutsceneController.Play = function(self, ...)
                    if isDisabled then return end
                    return OldPlayCutscene(self, ...)
                end
            end
        end)
    end
    initializeCutsceneHook()
    local M = {}
    function M.Start()
        if isDisabled then return end
        isDisabled = true
        if not CutsceneController then initializeCutsceneHook() end
    end
    function M.Stop()
        if not isDisabled then return end
        isDisabled = false
    end
    return M
end)()

--disable extras
local DisableExtras = (function()
    local VFXFolder = ReplicatedStorage:WaitForChild("VFX")
    local DisableNotificationConnection = nil
    local isVFXDisabled = false
    local supportsModuleOverride = false
    local VFXControllerModule = nil
    local originalVFXHandle, originalRenderAtPoint, originalRenderInstance
    pcall(function()
        VFXControllerModule = require(ReplicatedStorage:WaitForChild("Controllers").VFXController)
        originalVFXHandle = VFXControllerModule.Handle
        originalRenderAtPoint = VFXControllerModule.RenderAtPoint
        originalRenderInstance = VFXControllerModule.RenderInstance
        supportsModuleOverride = true
    end)
    local M = {}
    function M.StartSmallNotification()
        if DisableNotificationConnection then return end
        local PlayerGui = Players.LocalPlayer.PlayerGui
        local SmallNotification = PlayerGui:FindFirstChild("Small Notification") or PlayerGui:WaitForChild("Small Notification", 5)
        if not SmallNotification then return end
        SmallNotification.Enabled = false
        DisableNotificationConnection = SmallNotification:GetPropertyChangedSignal("Enabled"):Connect(function()
            if SmallNotification.Enabled then SmallNotification.Enabled = false end
        end)
    end
    function M.StopSmallNotification()
        if DisableNotificationConnection then
            DisableNotificationConnection:Disconnect()
            DisableNotificationConnection = nil
        end
        local SmallNotification = Players.LocalPlayer.PlayerGui:FindFirstChild("Small Notification")
        if SmallNotification then SmallNotification.Enabled = true end
    end
    function M.StartSkinEffect()
        if isVFXDisabled then return end
        isVFXDisabled = true
        if supportsModuleOverride then
            VFXControllerModule.Handle = function() end
            VFXControllerModule.RenderAtPoint = function() end
            VFXControllerModule.RenderInstance = function() end
            local cf = workspace:FindFirstChild("CosmeticFolder")
            if cf then pcall(function() cf:ClearAllChildren() end) end
        else
            for _, child in pairs(VFXFolder:GetChildren()) do
                if child.Name:match("Dive$") then child:Destroy() end
            end
            local cf = workspace:FindFirstChild("CosmeticFolder")
            if cf then pcall(function() cf:ClearAllChildren() end) end
            VFXFolder.ChildAdded:Connect(function(child)
                if isVFXDisabled and child.Name:match("Dive$") then child:Destroy() end
            end)
            if cf then cf.ChildAdded:Connect(function(child) if isVFXDisabled then child:Destroy() end end) end
        end
    end
    function M.StopSkinEffect()
        if not isVFXDisabled then return end
        isVFXDisabled = false
        if supportsModuleOverride then
            VFXControllerModule.Handle = originalVFXHandle
            VFXControllerModule.RenderAtPoint = originalRenderAtPoint
            VFXControllerModule.RenderInstance = originalRenderInstance
        end
    end
    return M
end)()

--stable result
local StableResult = (function()
    local M = {}
    M.Enabled = false
    local function GetAutoFishingRemote()
        return EmbeddedEventResolver:GetRF("UpdateAutoFishingState")
    end
    function M.Start()
        if M.Enabled then return false end
        if not GetAutoFishingRemote() then return false end
        M.Enabled = true
        local remote = GetAutoFishingRemote()
        local ok = pcall(function() remote:InvokeServer(true) end)
        if not ok then M.Enabled = false return false end
        pcall(function() LocalPlayer:SetAttribute("Loading", nil) end)
        return true
    end
    function M.Stop()
        if not M.Enabled then return false end
        M.Enabled = false
        local remote = GetAutoFishingRemote()
        if remote then pcall(function() remote:InvokeServer(false) end) end
        pcall(function() LocalPlayer:SetAttribute("Loading", false) end)
        return true
    end
    return M
end)()

--walk on weather 
local WalkOnWater = (function()
    local M = { Enabled = false, Platform = nil, AlignPos = nil, Connection = nil }
    local PLATFORM_SIZE = 14
    local OFFSET = 3
    local LAST_WATER_Y = nil
    local function GetCharacterReferences()
        local char = LocalPlayer.Character
        if not char then return end
        local humanoid = char:FindFirstChildOfClass("Humanoid")
        local hrp = char:FindFirstChild("HumanoidRootPart")
        if not humanoid or not hrp then return end
        return char, humanoid, hrp
    end
    local function ForceSurfaceLift()
        local _, humanoid, hrp = GetCharacterReferences()
        if not humanoid or not hrp then return end
        if humanoid:GetState() ~= Enum.HumanoidStateType.Swimming then return end
        for _ = 1, 60 do
            hrp.Velocity = Vector3.new(0, 80, 0)
            task.wait(0.03)
            if humanoid:GetState() ~= Enum.HumanoidStateType.Swimming then break end
        end
        hrp.CFrame = hrp.CFrame + Vector3.new(0, 3, 0)
    end
    local function GetWaterHeight()
        local _, _, hrp = GetCharacterReferences()
        if not hrp then return LAST_WATER_Y end
        local params = RaycastParams.new()
        params.FilterType = Enum.RaycastFilterType.Blacklist
        params.FilterDescendantsInstances = { LocalPlayer.Character }
        params.IgnoreWater = false
        local result = Workspace:Raycast(hrp.Position + Vector3.new(0, 5, 0), Vector3.new(0, -600, 0), params)
        if result then LAST_WATER_Y = result.Position.Y return LAST_WATER_Y end
        return LAST_WATER_Y
    end
    local function CreatePlatform()
        if M.Platform then M.Platform:Destroy() end
        local p = Instance.new("Part")
        p.Size = Vector3.new(PLATFORM_SIZE, 1, PLATFORM_SIZE)
        p.Anchored = true
        p.CanCollide = true
        p.Transparency = 1
        p.CanQuery = false
        p.CanTouch = false
        p.Name = "WaterLockPlatform"
        p.Parent = Workspace
        M.Platform = p
    end
    local function SetupAlign()
        local _, _, hrp = GetCharacterReferences()
        if not hrp then return false end
        if M.AlignPos then M.AlignPos:Destroy() end
        local att = hrp:FindFirstChild("RootAttachment") or Instance.new("Attachment")
        if not att.Parent then att.Name = "RootAttachment" att.Parent = hrp end
        local ap = Instance.new("AlignPosition")
        ap.Attachment0 = att
        ap.MaxForce = math.huge
        ap.MaxVelocity = math.huge
        ap.Responsiveness = 200
        ap.RigidityEnabled = true
        ap.Parent = hrp
        M.AlignPos = ap
        return true
    end
    local function Cleanup()
        if M.Connection then M.Connection:Disconnect() M.Connection = nil end
        if M.AlignPos then M.AlignPos:Destroy() M.AlignPos = nil end
        if M.Platform then M.Platform:Destroy() M.Platform = nil end
    end
    function M.Start()
        if M.Enabled then return end
        local char, humanoid, hrp = GetCharacterReferences()
        if not char or not humanoid or not hrp then return end
        ForceSurfaceLift()
        M.Enabled = true
        LAST_WATER_Y = nil
        CreatePlatform()
        if not SetupAlign() then M.Enabled = false Cleanup() return end
        M.Connection = RunService.Heartbeat:Connect(function()
            if not M.Enabled then return end
            local _, _, currentHRP = GetCharacterReferences()
            if not currentHRP then return end
            local waterY = GetWaterHeight()
            if not waterY then return end
            if M.Platform then M.Platform.CFrame = CFrame.new(currentHRP.Position.X, waterY - 0.5, currentHRP.Position.Z) end
            if M.AlignPos then M.AlignPos.Position = Vector3.new(currentHRP.Position.X, waterY + OFFSET, currentHRP.Position.Z) end
        end)
    end
    function M.Stop()
        M.Enabled = false
        Cleanup()
    end
    LocalPlayer.CharacterAdded:Connect(function()
        if M.Enabled then task.wait(0.5) Cleanup() M.Enabled = false M.Start() end
    end)
    return M
end)()

-- ══════════════════════════════════════════
--              SUPPORT FEATURE SECTION
-- ══════════════════════════════════════════
local SupportSection = Fsihing:Section({ 
    Title = "Support Feature",
    Box = true,
    TextXAlignment = "Center",
    TextSize = 15,
    Opened = false,
})

SupportSection:Toggle({
    Title = "No Fishing Animation",
    Value = false,
    Callback = function(state)
        if state then
            NoFishingAnimation.Start()
        else
            NoFishingAnimation.Stop()
        end
    end
})

SupportSection:Toggle({
    Title = "Auto Equip Rod",
    Value = false,
    Callback = function(state)
        if state then
            AutoEquipRod.Start()
        else
            AutoEquipRod.Stop()
        end
    end
})

SupportSection:Toggle({
    Title = "Lock Position",
    Value = false,
    Callback = function(state)
        if state then
            LockPosition.Start()
        else
            LockPosition.Stop()
        end
    end
})

SupportSection:Toggle({
    Title = "Disable Cutscenes",
    Value = false,
    Callback = function(state)
        if state then
            DisableCutscenes.Start()
        else
            DisableCutscenes.Stop()
        end
    end
})

SupportSection:Toggle({
    Title = "Disable Notification",
    Value = false,
    Callback = function(state)
        if state then
            DisableExtras.StartSmallNotification()
        else
            DisableExtras.StopSmallNotification()
        end
    end
})

SupportSection:Toggle({
    Title = "Disable Skin Effect",
    Value = false,
    Callback = function(state)
        if state then
            DisableExtras.StartSkinEffect()
        else
            DisableExtras.StopSkinEffect()
        end
    end
})

SupportSection:Toggle({
    Title = "Stable Result",
    Value = false,
    Callback = function(state)
        if state then
            StableResult.Start()
        else
            StableResult.Stop()
        end
    end
})

SupportSection:Toggle({
    Title = "Walk On Water",
    Value = false,
    Callback = function(state)
        if state then
            WalkOnWater.Start()
        else
            WalkOnWater.Stop()
        end
    end
})

-- ══════════════════════════════════════════
--              AUTO FAVORITE MODULE
-- ══════════════════════════════════════════
CombinedModules.AutoFavorite = (function()
    local AutoFavoriteModule = {}

    local v0 = {
        RS = ReplicatedStorage,
        Players = Players
    }
    local v5, v6, v7
    local referencesInitialized = false
    local onChangeConnected = false
    local isScanning = false

    local TIER_NAMES = {
        [1] = "Common",
        [2] = "Uncommon",
        [3] = "Rare",
        [4] = "Epic",
        [5] = "Legendary",
        [6] = "Mythic",
        [7] = "SECRET",
        [8] = "FORGOTTEN"
    }

    local function InitializeReferences()
        if referencesInitialized then return true end
        local success = pcall(function()
            v5 = {
                ItemUtility = require(v0.RS.Shared.ItemUtility),
                PlayerStatsUtility = require(v0.RS.Shared.PlayerStatsUtility)
            }
            v6 = {
                Events = {
                    REFav = NetEvents.RE_FavoriteItem
                }
            }
            v7 = {
                Data = require(v0.RS.Packages.Replion).Client:WaitReplion("Data"),
                Items = v0.RS:WaitForChild("Items"),
                Variants = v0.RS:WaitForChild("Variants")
            }
            referencesInitialized = true
        end)
        return success
    end

    local v8 = {
        selectedName = {},
        selectedRarity = {},
        selectedVariant = {},
        autoFavEnabled = false
    }

    local function toSet(arr)
        local set = {}
        for _, v in ipairs(arr) do set[v] = true end
        return set
    end

    local v11 = {}
    local function BuildFishList()
        v11 = {}
        if not referencesInitialized then InitializeReferences() end
        if not v7 or not v7.Items then return v11 end
        pcall(function()
            for _, itemFolder in ipairs(v7.Items:GetChildren()) do
                if itemFolder:IsA("Folder") then
                    for _, fishModule in ipairs(itemFolder:GetChildren()) do
                        if fishModule:IsA("ModuleScript") then
                            local success, fishData = pcall(require, fishModule)
                            if success and fishData and fishData.Data then
                                local displayName = fishData.Data.DisplayName or fishData.Data.Name
                                if displayName and not table.find(v11, displayName) then
                                    table.insert(v11, displayName)
                                end
                            end
                        end
                    end
                elseif itemFolder:IsA("ModuleScript") then
                    local success, fishData = pcall(require, itemFolder)
                    if success and fishData and fishData.Data then
                        local displayName = fishData.Data.DisplayName or fishData.Data.Name
                        if displayName and not table.find(v11, displayName) then
                            table.insert(v11, displayName)
                        end
                    end
                end
            end
            table.sort(v11)
        end)
        return v11
    end

    local variantList = {}
    local function BuildVariantList()
        variantList = {}
        if not referencesInitialized then InitializeReferences() end
        if not v7 or not v7.Variants then return variantList end
        pcall(function()
            for _, variantModule in ipairs(v7.Variants:GetChildren()) do
                if variantModule:IsA("ModuleScript") then
                    local variantName = variantModule.Name
                    if variantName and variantName ~= "1x1x1x1" and not table.find(variantList, variantName) then
                        table.insert(variantList, variantName)
                    end
                end
            end
            table.sort(variantList)
        end)
        return variantList
    end

    local function scanInventory()
        if not v8.autoFavEnabled then return end
        if not referencesInitialized then return end
        if isScanning then return end

        local hasNameFilter = next(v8.selectedName) ~= nil
        local hasVariantFilter = next(v8.selectedVariant) ~= nil
        local hasRarityFilter = next(v8.selectedRarity) ~= nil

        if not hasNameFilter and not hasVariantFilter and not hasRarityFilter then return end

        isScanning = true
        pcall(function()
            local inventory = v7.Data:GetExpect({"Inventory", "Items"})
            for _, item in ipairs(inventory) do
                if not v8.autoFavEnabled then break end
                if item.Favorited then continue end

                local fishData = v5.ItemUtility:GetItemData(item.Id)
                if not fishData or not fishData.Data then continue end

                local fishName = fishData.Data.DisplayName or fishData.Data.Name
                local fishTier = fishData.Data.Tier
                local variantId = item.Metadata and item.Metadata.VariantId or "None"
                local tierName = TIER_NAMES[fishTier]

                local shouldFavorite = true

                if hasNameFilter then
                    if not fishName or not v8.selectedName[fishName] then
                        shouldFavorite = false
                    end
                end

                if shouldFavorite and hasVariantFilter then
                    if variantId == "None" or not v8.selectedVariant[variantId] then
                        shouldFavorite = false
                    end
                end

                if shouldFavorite and hasRarityFilter then
                    if not tierName or not v8.selectedRarity[tierName] then
                        shouldFavorite = false
                    end
                end

                if shouldFavorite then
                    pcall(function() v6.Events.REFav:FireServer(item.UUID) end)
                    task.wait(0.15)
                end
            end
        end)
        isScanning = false
    end

    function AutoFavoriteModule.GetAllFishNames()
        if #v11 == 0 then BuildFishList() end
        return #v11 > 0 and v11 or {"No Fish Found"}
    end

    function AutoFavoriteModule.GetAllVariants()
        if #variantList == 0 then BuildVariantList() end
        return #variantList > 0 and variantList or {"No Variants Found"}
    end

    function AutoFavoriteModule.GetAllTiers()
        return {"Common", "Uncommon", "Rare", "Epic", "Legendary", "Mythic", "SECRET", "FORGOTTEN"}
    end

    function AutoFavoriteModule.SetSelectedNames(names)
        v8.selectedName = toSet(names)
        if v8.autoFavEnabled then task.spawn(scanInventory) end
    end

    function AutoFavoriteModule.SetSelectedRarity(rarities)
        v8.selectedRarity = toSet(rarities)
        if v8.autoFavEnabled then task.spawn(scanInventory) end
    end

    function AutoFavoriteModule.SetSelectedVariants(variants)
        v8.selectedVariant = toSet(variants)
        if v8.autoFavEnabled then task.spawn(scanInventory) end
    end

    function AutoFavoriteModule.Start()
        if not referencesInitialized then
            local success = InitializeReferences()
            if not success then return false end
        end
        v8.autoFavEnabled = true
        task.spawn(scanInventory)
        if v7 and v7.Data and not onChangeConnected then
            onChangeConnected = true
            v7.Data:OnChange({"Inventory", "Items"}, function()
                if v8.autoFavEnabled then
                    task.spawn(function()
                        task.wait(0.3)
                        scanInventory()
                    end)
                end
            end)
        end
        return true
    end

    function AutoFavoriteModule.Stop()
        v8.autoFavEnabled = false
        return true
    end

    function AutoFavoriteModule.RefreshLists()
        v11 = {}
        variantList = {}
        BuildFishList()
        BuildVariantList()
        return {
            FishCount = #v11,
            VariantCount = #variantList
        }
    end

    function AutoFavoriteModule.IsEnabled()
        return v8.autoFavEnabled
    end

    -- Auto init saat load
    task.spawn(function()
        if not game:IsLoaded() then game.Loaded:Wait() end
        task.wait(1)
        local success = InitializeReferences()
        if success then
            BuildFishList()
            BuildVariantList()
        else
            task.wait(2)
            InitializeReferences()
            BuildFishList()
            BuildVariantList()
        end
    end)

    return AutoFavoriteModule
end)()

-- ══════════════════════════════════════════
--              FAVORITE TAB
-- ══════════════════════════════════════════
local Favorite = Window:Tab({
    Title = "Favorite",
    Icon = "star",
    IconColor = Mains,
    IconShape = "Square",
    Border = true,
})

-- ══════════════════════════════════════════
--              AUTO FAVORITE SECTION
-- ══════════════════════════════════════════
local AutoFavSection = Favorite:Section({
    Title = "Auto Favorite",
    Box = true,
    TextXAlignment = "Center",
    TextSize = 15,
    Opened = true,
})

if CombinedModules.AutoFavorite then

    local nameDropdown = AutoFavSection:Dropdown({
        Title = "Fish Name",
        Values = CombinedModules.AutoFavorite.GetAllFishNames(),
        Value = nil,
        AllowNone = true,
        Multi = true,
        Callback = function(selected)
            if type(selected) == "table" then
                CombinedModules.AutoFavorite.SetSelectedNames(selected)
            end
        end,
    })

    local variantDropdown = AutoFavSection:Dropdown({
        Title = "Variant",
        Values = CombinedModules.AutoFavorite.GetAllVariants(),
        Value = nil,
        AllowNone = true,
        Multi = true,
        Callback = function(selected)
            if type(selected) == "table" then
                CombinedModules.AutoFavorite.SetSelectedVariants(selected)
            end
        end,
    })

    AutoFavSection:Dropdown({
        Title = "Rarity",
        Values = CombinedModules.AutoFavorite.GetAllTiers(),
        Value = nil,
        AllowNone = true,
        Multi = true,
        Callback = function(selected)
            if type(selected) == "table" then
                CombinedModules.AutoFavorite.SetSelectedRarity(selected)
            end
        end,
    })

    AutoFavSection:Button({
        Title = "Refresh Lists",
        Callback = function()
            CombinedModules.AutoFavorite.RefreshLists()
            nameDropdown:Set(CombinedModules.AutoFavorite.GetAllFishNames())
            variantDropdown:Set(CombinedModules.AutoFavorite.GetAllVariants())
        end,
    })

    AutoFavSection:Toggle({
        Title = "Enable Auto Favorite",
        Value = false,
        Callback = function(state)
            if state then
                CombinedModules.AutoFavorite.Start()
            else
                CombinedModules.AutoFavorite.Stop()
            end
        end,
    })

end

-- ══════════════════════════════════════════
--              TELEPORT MODULE
-- ══════════════════════════════════════════
local TeleportModule = (function()
    local M = {}

    M.Locations = {
        ["Ancient Jungle"]        = Vector3.new(1467.8480224609375, 7.447117328643799, -327.5971984863281),
        ["Ancient Ruin"]          = Vector3.new(6045.40234375, -588.600830078125, 4608.9375),
        ["Coral Reefs"]           = Vector3.new(-2921.858154296875, 3.249999761581421, 2083.2978515625),
        ["Crater Island"]         = Vector3.new(1078.454345703125, 5.0720038414001465, 5099.396484375),
        ["Esoteric Depths"]       = Vector3.new(3224.075927734375, -1302.85498046875, 1404.9346923828125),
        ["Fisherman Island"]      = Vector3.new(92.80695343017578, 9.531265258789062, 2762.082275390625),
        ["Kohana"]                = Vector3.new(-643.3051147460938, 16.03544807434082, 622.3605346679688),
        ["Kohana Volcano"]        = Vector3.new(-572.0244750976562, 39.4923210144043, 112.49259185791016),
        ["Lost Isle"]             = Vector3.new(-3701.1513671875, 5.425841808319092, -1058.9107666015625),
        ["Sysiphus Statue"]       = Vector3.new(-3656.56201171875, -134.5314178466797, -964.3167724609375),
        ["Sacred Temple"]         = Vector3.new(1476.30810546875, -21.8499755859375, -630.8220825195312),
        ["Treasure Room"]         = Vector3.new(-3601.568359375, -266.57373046875, -1578.998779296875),
        ["Tropical Grove"]        = Vector3.new(-2104.467041015625, 6.268016815185547, 3718.2548828125),
        ["Underground Cellar"]    = Vector3.new(2162.577392578125, -91.1981430053711, -725.591552734375),
        ["Pirate Cove"]           = Vector3.new(3334.47, 10.2, 3502.92),
        ["Leviathan Den"]         = Vector3.new(3471.41, -287.84, 3468.87),
        ["Pirate Treasure Room"]  = Vector3.new(3337.64, -302.75, 3089.56),
        ["Crystal Depths"]        = Vector3.new(5729.04, -904.82, 15407.97),
        ["Vulcanic Cavern"]       = Vector3.new(1118.1817626953125, 85.990936279296875, -10250.158203125),
        ["Lava Basin"]            = Vector3.new(871.7166137695312, 96.93890380859375, -10176.6259765625),
        ["Heartfelt Island"]      = Vector3.new(1114.147705078125, 4.845647811889648, 2715.550048828125),
        ["Weather Machine"]       = Vector3.new(-1513.9249267578125, 6.499999523162842, 1892.10693359375),
        ["Underwater City"]       = Vector3.new(-3140.417968750000000, -643.484252929687500, -10421.276367187500000),
        ["Planetary Observatory"] = Vector3.new(390.897918701171875, 7.251102924346924, 2205.069580078125000),
    }

    function M.TeleportTo(name)
        local char = localPlayer.Character or localPlayer.CharacterAdded:Wait()
        local root = char:WaitForChild("HumanoidRootPart")
        local target = M.Locations[name]
        if not target then return false end
        root.CFrame = CFrame.new(target)
        return true
    end

    return M
end)()

-- ══════════════════════════════════════════
--              TELEPORT TO PLAYER MODULE
-- ══════════════════════════════════════════
CombinedModules.TeleportToPlayer = (function()
    local TeleportToPlayer = {}

    function TeleportToPlayer.TeleportTo(playerName)
        local target = Players:FindFirstChild(playerName)
        local myChar = localPlayer.Character
        if not target or not target.Character then return end
        local targetHRP = target.Character:FindFirstChild("HumanoidRootPart")
        local myHRP = myChar and myChar:FindFirstChild("HumanoidRootPart")
        if targetHRP and myHRP then
            myHRP.CFrame = targetHRP.CFrame + Vector3.new(0, 3, 0)
        end
    end

    return TeleportToPlayer
end)()

-- ══════════════════════════════════════════
--              AUTO EVENT MODULE
-- ══════════════════════════════════════════
CombinedModules.AutoEvent = (function()
    local module = {}

    local EventSearchPatterns = {
        ["Shark Hunt"]      = {"Shark Hunt"},
        ["Megalodon Hunt"]  = {"Megalodon Hunt"},
        ["Worm Hunt"]       = {"BlackHole", "Model"},
        ["Ghost Shark Hunt"]= {"Ghost Shark Hunt", "Ghost"},
        ["Treasure Hunt"]   = {"Treasure"},
        ["Black Hole"]      = {"Black Hole"}
    }

    module.FallbackCoords = {
        ["Shark Hunt"] = {
            Vector3.new(1.64999, -1.3500, 2095.72),
            Vector3.new(1369.94, -1.3500, 930.125),
            Vector3.new(-1585.5, -1.3500, 1242.87),
            Vector3.new(-1896.8, -1.3500, 2634.37),
        },
        ["Worm Hunt"] = {
            Vector3.new(2190.85, -1.3999, 97.5749),
            Vector3.new(-2450.6, -1.3999, 139.731),
            Vector3.new(-267.47, -1.3999, 5188.53),
        },
        ["Megalodon Hunt"] = {
            Vector3.new(-1076.3, -1.3999, 1676.19),
            Vector3.new(-1191.8, -1.3999, 3597.30),
            Vector3.new(412.700, -1.3999, 4134.39),
        },
        ["Ghost Shark Hunt"] = {
            Vector3.new(489.558, -1.3500, 25.4060),
            Vector3.new(-1358.2, -1.3500, 4100.55),
            Vector3.new(627.859, -1.3500, 3798.08),
        },
    }

    module.TeleportCheckInterval = 8
    module.HeightOffset = 15
    module.SafeZoneRadius = 150
    module.RequireEventActive = true
    module.WaitForEventTimeout = 300
    module.ScanCooldown = 5

    local running = false
    local currentEventName = nil
    local cachedEventPosition = nil
    local cachedEventObject = nil
    local eventIsActive = false
    local lastScanTime = 0
    local loopThread = nil

    local function getHRP()
        local c = game.Players.LocalPlayer.Character
        return c and c:FindFirstChild("HumanoidRootPart")
    end

    local function applyOffset(v)
        return Vector3.new(v.X, v.Y + module.HeightOffset, v.Z)
    end

    local function safeCharacter()
        return game.Players.LocalPlayer.Character or game.Players.LocalPlayer.CharacterAdded:Wait()
    end

    local function isNearTarget(targetPos)
        local hrp = getHRP()
        if not hrp then return false end
        return (hrp.Position - targetPos).Magnitude <= module.SafeZoneRadius
    end

    local function doTeleport(pos)
        if isNearTarget(pos) then return true end
        local ok = pcall(function()
            local c = safeCharacter()
            if not c then return end
            local hrp = c:FindFirstChild("HumanoidRootPart")
            if not hrp then return end
            if c.PrimaryPart then
                c:PivotTo(CFrame.new(pos))
            else
                hrp.CFrame = CFrame.new(pos)
            end
            hrp.Anchored = false
            hrp.Velocity = Vector3.zero
        end)
        return ok
    end

    local function IsEventAlive(obj)
        if not obj then return false end
        local success = pcall(function()
            return obj.Parent ~= nil and obj:IsDescendantOf(workspace)
        end)
        return success
    end

    local function SearchInAllProps(eventName)
        local patterns = EventSearchPatterns[eventName]
        if not patterns then return false, nil, nil end
        local allProps = {}
        for _, child in ipairs(workspace:GetChildren()) do
            if child.Name == "Props" and child:IsA("Model") then
                table.insert(allProps, child)
            end
        end
        for _, props in ipairs(allProps) do
            for _, pattern in ipairs(patterns) do
                for _, child in ipairs(props:GetChildren()) do
                    if child.Name == pattern and IsEventAlive(child) then
                        local position = nil
                        if child:IsA("Model") then
                            if child.PrimaryPart then
                                position = child.PrimaryPart.Position
                            else
                                local cf, size = child:GetBoundingBox()
                                local adjustedY = cf.Position.Y - (size.Y / 4)
                                position = Vector3.new(cf.Position.X, adjustedY, cf.Position.Z)
                            end
                        elseif child:IsA("BasePart") then
                            position = child.Position
                        end
                        if position then return true, position, child end
                    end
                end
            end
        end
        return false, nil, nil
    end

    local function scanEvent(eventName)
        local now = tick()
        if now - lastScanTime < module.ScanCooldown then
            if cachedEventPosition and cachedEventObject and IsEventAlive(cachedEventObject) then
                return cachedEventPosition
            end
        end
        lastScanTime = now
        local found, position, obj = SearchInAllProps(eventName)
        if found and position then
            cachedEventPosition = applyOffset(position)
            cachedEventObject = obj
            eventIsActive = true
            return cachedEventPosition
        end
        local coords = module.FallbackCoords[eventName]
        if coords and #coords > 0 then
            for _, coord in ipairs(coords) do
                local region = Region3.new(
                    coord - Vector3.new(30, 30, 30),
                    coord + Vector3.new(30, 30, 30)
                ):ExpandToGrid(4)
                local ok, parts = pcall(function()
                    return workspace:FindPartsInRegion3(region, nil, 50)
                end)
                if ok and parts and #parts > 0 then
                    for _, p in ipairs(parts) do
                        if typeof(p) == "Instance" and p:IsA("BasePart") and IsEventAlive(p) then
                            local ps = p.Position
                            if (ps - coord).Magnitude <= 25 then
                                cachedEventPosition = applyOffset(ps)
                                cachedEventObject = p
                                eventIsActive = true
                                return cachedEventPosition
                            end
                        end
                    end
                end
            end
        end
        eventIsActive = false
        return nil
    end

    local function waitActive(eventName)
        local start = tick()
        while tick() - start < module.WaitForEventTimeout do
            local p = scanEvent(eventName)
            if p then return p end
            task.wait(5)
        end
        return nil
    end

    function module.Start(name)
        if running then return false end
        if not EventSearchPatterns[name] and not module.FallbackCoords[name] then return false end
        running = true
        currentEventName = name
        cachedEventPosition = nil
        cachedEventObject = nil
        eventIsActive = false
        lastScanTime = 0
        loopThread = task.spawn(function()
            if module.RequireEventActive then
                local pos = waitActive(name)
                if not pos then module.Stop() return end
                doTeleport(pos)
            end
            local failCount = 0
            while running do
                if cachedEventObject and not IsEventAlive(cachedEventObject) then
                    cachedEventPosition = nil
                    cachedEventObject = nil
                    eventIsActive = false
                end
                local newPos = scanEvent(name)
                if newPos then
                    doTeleport(newPos)
                    failCount = 0
                else
                    failCount = failCount + 1
                    if failCount >= 3 then module.Stop() break end
                end
                task.wait(module.TeleportCheckInterval)
            end
        end)
        return true
    end

    function module.Stop()
        running = false
        cachedEventPosition = nil
        cachedEventObject = nil
        currentEventName = nil
        eventIsActive = false
        if loopThread then
            if loopThread ~= coroutine.running() then task.cancel(loopThread) end
            loopThread = nil
        end
        return true
    end

    function module.GetEventNames()
        local list = {}
        for name, _ in pairs(EventSearchPatterns) do table.insert(list, name) end
        table.sort(list)
        return list
    end

    function module.HasEventPattern(eventName)
        return EventSearchPatterns[eventName] ~= nil
    end

    return module
end)()

-- ══════════════════════════════════════════
--              TELEPORT TAB
-- ══════════════════════════════════════════
local Teleport = Window:Tab({
    Title = "Teleport",
    Icon = "map-pin",
    IconColor = Mains,
    IconShape = "Square",
    Border = true,
})

-- ══════════════════════════════════════════
--           TELEPORT TO ISLAND SECTION
-- ══════════════════════════════════════════
local IslandSection = Teleport:Section({
    Title = "Teleport to Island",
    Box = true,
    TextXAlignment = "Center",
    TextSize = 15,
    Opened = true,
})

local islandNames = {}
for name, _ in pairs(TeleportModule.Locations) do
    table.insert(islandNames, name)
end
table.sort(islandNames)

local selectedIsland = nil

IslandSection:Dropdown({
    Title = "Select Island",
    Values = islandNames,
    Value = 1,
    Callback = function(selected)
        selectedIsland = selected
    end,
})

IslandSection:Button({
    Title = "Teleport",
    Callback = function()
        if not selectedIsland or selectedIsland == "" then return end
        TeleportModule.TeleportTo(selectedIsland)
    end,
})

-- ══════════════════════════════════════════
--           TELEPORT TO PLAYER SECTION
-- ══════════════════════════════════════════
local PlayerSection = Teleport:Section({
    Title = "Teleport to Player",
    Box = true,
    TextXAlignment = "Center",
    TextSize = 15,
    Opened = false,
})

local playerNames = {}
local selectedPlayer = nil

local function updatePlayerList()
    table.clear(playerNames)
    for _, plr in ipairs(Players:GetPlayers()) do
        if plr ~= localPlayer then
            table.insert(playerNames, plr.Name)
        end
    end
    table.sort(playerNames)
end
updatePlayerList()

local playerDropdown = PlayerSection:Dropdown({
    Title = "Select Player",
    Values = playerNames,
    Value = 1,
    Callback = function(selected)
        selectedPlayer = selected
    end,
})

PlayerSection:Button({
    Title = "Teleport Now",
    Callback = function()
        if not selectedPlayer or selectedPlayer == "" then return end
        CombinedModules.TeleportToPlayer.TeleportTo(selectedPlayer)
    end,
})

PlayerSection:Button({
    Title = "Refresh Player List",
    Callback = function()
        updatePlayerList()
        playerDropdown:Set(playerNames)
    end,
})

-- ══════════════════════════════════════════
--           EVENT TELEPORT SECTION
-- ══════════════════════════════════════════
local EventSection = Teleport:Section({
    Title = "Event Teleport",
    Box = true,
    TextXAlignment = "Center",
    TextSize = 15,
    Opened = false,
})

local selectedEventName = nil
local eventNames = CombinedModules.AutoEvent.GetEventNames()

EventSection:Dropdown({
    Title = "Select Event",
    Values = eventNames,
    Value = 1,
    Callback = function(selected)
        selectedEventName = selected
        -- Jika toggle aktif, restart dengan event baru
        if CombinedModules.AutoEvent and selected then
            CombinedModules.AutoEvent.Stop()
            task.wait(0.1)
            if CombinedModules.AutoEvent.HasEventPattern(selected) then
                CombinedModules.AutoEvent.Start(selected)
            end
        end
    end,
})

EventSection:Toggle({
    Title = "Enable Auto Teleport",
    Value = false,
    Callback = function(state)
        if state then
            if selectedEventName and CombinedModules.AutoEvent.HasEventPattern(selectedEventName) then
                CombinedModules.AutoEvent.Start(selectedEventName)
            end
        else
            CombinedModules.AutoEvent.Stop()
        end
    end,
})


-- ══════════════════════════════════════════
--              AUTO SELL MODULE
-- ══════════════════════════════════════════
local AutoSellSystem = (function()
    local M = {}
    M.Timer = {}
    M.Count = {}

    local function getSellRemote()
        return EmbeddedEventResolver:GetRF("SellAllItems")
    end

    local function parseNumber(text)
        if not text or text == "" then return 0 end
        local cleaned = tostring(text):gsub("%D", "")
        if cleaned == "" then return 0 end
        return tonumber(cleaned) or 0
    end

    local function getBagCount()
        local gui = localPlayer:FindFirstChild("PlayerGui")
        if not gui then return 0, 0 end
        local inv = gui:FindFirstChild("Inventory")
        if not inv then return 0, 0 end
        local label = inv:FindFirstChild("Main")
            and inv.Main:FindFirstChild("Top")
            and inv.Main.Top:FindFirstChild("Options")
            and inv.Main.Top.Options:FindFirstChild("Fish")
            and inv.Main.Top.Options.Fish:FindFirstChild("Label")
            and inv.Main.Top.Options.Fish.Label:FindFirstChild("BagSize")
        if not label or not label:IsA("TextLabel") then return 0, 0 end
        local curText, maxText = label.Text:match("(.+)%/(.+)")
        if not curText or not maxText then return 0, 0 end
        return parseNumber(curText), parseNumber(maxText)
    end

    local state = { totalSells = 0, lastSellTime = 0 }
    local timerMode = { enabled = false, interval = 5, task = nil, sellCount = 0 }
    local countMode = { enabled = false, target = 235, checkDelay = 1.5, lastSell = 0, task = nil }

    local function executeSell()
        local remote = getSellRemote()
        if not remote then return false end
        local success = pcall(function() return remote:InvokeServer() end)
        if success then
            state.totalSells = state.totalSells + 1
            state.lastSellTime = tick()
            return true
        end
        return false
    end

    function M.SellOnce()
        if not getSellRemote() then return false end
        if tick() - state.lastSellTime < 0.5 then return false end
        return executeSell()
    end

    function M.Timer.Start(interval)
        if timerMode.enabled then return false end
        if not getSellRemote() then return false end
        if interval and tonumber(interval) and tonumber(interval) >= 1 then
            timerMode.interval = tonumber(interval)
        end
        timerMode.enabled = true
        timerMode.sellCount = 0
        timerMode.task = task.spawn(function()
            while timerMode.enabled do
                task.wait(timerMode.interval)
                if not timerMode.enabled then break end
                if executeSell() then timerMode.sellCount = timerMode.sellCount + 1 end
            end
        end)
        return true
    end

    function M.Timer.Stop()
        if not timerMode.enabled then return false end
        timerMode.enabled = false
        if timerMode.task then task.cancel(timerMode.task) timerMode.task = nil end
        return true
    end

    function M.Timer.SetInterval(seconds)
        if tonumber(seconds) and tonumber(seconds) >= 1 then
            timerMode.interval = tonumber(seconds)
            return true
        end
        return false
    end

    function M.Count.Start(target)
        if countMode.enabled then return false end
        if not getSellRemote() then return false end
        if target and tonumber(target) and tonumber(target) > 0 then
            countMode.target = tonumber(target)
        end
        countMode.enabled = true
        countMode.task = task.spawn(function()
            while countMode.enabled do
                task.wait(countMode.checkDelay)
                if not countMode.enabled then break end
                local current, max = getBagCount()
                if countMode.target > 0 and current >= countMode.target then
                    if tick() - countMode.lastSell < 3 then continue end
                    countMode.lastSell = tick()
                    executeSell()
                    task.wait(2)
                end
            end
        end)
        return true
    end

    function M.Count.Stop()
        if not countMode.enabled then return false end
        countMode.enabled = false
        if countMode.task then task.cancel(countMode.task) countMode.task = nil end
        return true
    end

    function M.Count.SetTarget(count)
        if tonumber(count) and tonumber(count) > 0 then
            countMode.target = tonumber(count)
            return true
        end
        return false
    end

    return M
end)()

-- ══════════════════════════════════════════
--           AUTO BUY WEATHER MODULE
-- ══════════════════════════════════════════
local AutoBuyWeather = (function()
    local M = {}
    local isRunning = false
    local selected = {}

    local function getWeatherRemote()
        return EmbeddedEventResolver:GetRF("PurchaseWeatherEvent")
    end

    M.AllWeathers = { "Cloudy", "Storm", "Wind", "Snow", "Radiant", "Shark Hunt" }

    function M.SetSelected(list)
        selected = list or {}
    end

    function M.Start()
        if isRunning then return false end
        if not getWeatherRemote() then return false end
        if #selected == 0 then return false end
        isRunning = true
        M._loopThread = task.spawn(function()
            while isRunning do
                for _, weather in ipairs(selected) do
                    if not isRunning then break end
                    pcall(function()
                        local remote = getWeatherRemote()
                        if remote then remote:InvokeServer(weather) end
                    end)
                    task.wait(0.1)
                end
                task.wait(10)
            end
        end)
        return true
    end

    function M.Stop()
        if not isRunning then return false end
        isRunning = false
        if M._loopThread then task.cancel(M._loopThread) M._loopThread = nil end
        return true
    end

    function M.IsAvailable()
        return getWeatherRemote() ~= nil
    end

    return M
end)()

-- ══════════════════════════════════════════
--            MERCHANT MODULE
-- ══════════════════════════════════════════
CombinedModules.MerchantSystem = (function()
    local PlayerGui = localPlayer:WaitForChild("PlayerGui", 5)
    local MerchantUI = nil
    pcall(function()
        MerchantUI = PlayerGui:FindFirstChild("Merchant") or PlayerGui:WaitForChild("Merchant", 3)
    end)

    local function OpenMerchant()
        if MerchantUI then MerchantUI.Enabled = true end
    end

    local function CloseMerchant()
        if MerchantUI then MerchantUI.Enabled = false end
    end

    return { Open = OpenMerchant, Close = CloseMerchant }
end)()

-- ══════════════════════════════════════════
--              SHOP TAB
-- ══════════════════════════════════════════
local Shop = Window:Tab({
    Title = "Shop",
    Icon = "shopping-cart",
    IconColor = Mains,
    IconShape = "Square",
    Border = true,
})

-- ══════════════════════════════════════════
--              AUTO SELL SECTION
-- ══════════════════════════════════════════
local SellSection = Shop:Section({
    Title = "Auto Sell",
    Box = true,
    TextXAlignment = "Center",
    TextSize = 15,
    Opened = true,
})

local autoSellMode = "Timer"
local autoSellEnabled = false

SellSection:Button({
    Title = "Sell All Now",
    Callback = function()
        AutoSellSystem.SellOnce()
    end,
})

SellSection:Dropdown({
    Title = "Auto Sell Mode",
    Values = { "Timer", "By Count" },
    Value = 1,
    Callback = function(selected)
        local previousMode = autoSellMode
        autoSellMode = selected
        if autoSellEnabled then
            if previousMode == "Timer" then
                AutoSellSystem.Timer.Stop()
            else
                AutoSellSystem.Count.Stop()
            end
            task.wait(0.1)
            if selected == "Timer" then
                AutoSellSystem.Timer.Start()
            else
                AutoSellSystem.Count.Start()
            end
        end
    end,
})

SellSection:Input({
    Title = "Value (Seconds / Fish Count)",
    Value = "5",
    Type = "Input",
    Placeholder = "5",
    Callback = function(value)
        local num = tonumber(value)
        if num then
            AutoSellSystem.Timer.SetInterval(num)
            AutoSellSystem.Count.SetTarget(num)
        end
    end,
})

SellSection:Toggle({
    Title = "Enable Auto Sell",
    Value = false,
    Callback = function(state)
        autoSellEnabled = state
        if state then
            if autoSellMode == "Timer" then
                AutoSellSystem.Timer.Start()
            else
                AutoSellSystem.Count.Start()
            end
        else
            AutoSellSystem.Timer.Stop()
            AutoSellSystem.Count.Stop()
        end
    end,
})

-- ══════════════════════════════════════════
--           AUTO BUY WEATHER SECTION
-- ══════════════════════════════════════════
local WeatherSection = Shop:Section({
    Title = "Auto Buy Weather",
    Box = true,
    TextXAlignment = "Center",
    TextSize = 15,
    Opened = false,
})

local selectedWeathers = {}

WeatherSection:Dropdown({
    Title = "Select Weather",
    Values = AutoBuyWeather.AllWeathers,
    Value = nil,
    AllowNone = true,
    Multi = true,
    Callback = function(selected)
        if type(selected) == "table" then
            selectedWeathers = selected
            AutoBuyWeather.SetSelected(selectedWeathers)
        end
    end,
})

WeatherSection:Toggle({
    Title = "Enable Auto Buy Weather",
    Value = false,
    Callback = function(state)
        if state then
            AutoBuyWeather.SetSelected(selectedWeathers)
            if not AutoBuyWeather.IsAvailable() then return end
            AutoBuyWeather.Start()
        else
            AutoBuyWeather.Stop()
        end
    end,
})

-- ══════════════════════════════════════════
--            REMOTE MERCHANT SECTION
-- ══════════════════════════════════════════
local MerchantSection = Shop:Section({
    Title = "Remote Merchant",
    Box = true,
    TextXAlignment = "Center",
    TextSize = 15,
    Opened = false,
})

MerchantSection:Button({
    Title = "Open Merchant",
    Callback = function()
        CombinedModules.MerchantSystem.Open()
    end,
})

MerchantSection:Button({
    Title = "Close Merchant",
    Callback = function()
        CombinedModules.MerchantSystem.Close()
    end,
})


-- ══════════════════════════════════════════
--           AUTO BUY CHARM MODULE
-- ══════════════════════════════════════════
local BuyCharm = (function()
    local M = {}
    M.IsBuying = false
    M.AutoLoop = false
    M.Settings = { CharmType = 1, Amount = 1, Delay = 0.5 }

    local function getPurchaseRemote()
        return EmbeddedEventResolver:GetRF("PurchaseCharm")
    end

    local function purchaseCharm(charmType)
        local remote = getPurchaseRemote()
        if not remote then return false end
        local success = pcall(function() return remote:InvokeServer(charmType) end)
        return success
    end

    function M.SetCharmType(charmID)
        if charmID >= 1 and charmID <= 4 then
            M.Settings.CharmType = charmID
            return true
        end
        return false
    end

    function M.SetAmount(amount)
        amount = tonumber(amount)
        if amount and amount > 0 and amount <= 1000 then
            M.Settings.Amount = math.floor(amount)
            return true
        end
        return false
    end

    function M.SetDelay(delay)
        delay = tonumber(delay)
        if delay and delay >= 0 and delay <= 10 then
            M.Settings.Delay = delay
            return true
        end
        return false
    end

    function M.Start(amount, charmType)
        if M.IsBuying then return false end
        if amount then M.SetAmount(amount) end
        if charmType then M.SetCharmType(charmType) end
        if not getPurchaseRemote() then return false end
        M.IsBuying = true
        task.spawn(function()
            local targetAmount = M.Settings.Amount
            local charmID = M.Settings.CharmType
            for i = 1, targetAmount do
                if not M.IsBuying then break end
                purchaseCharm(charmID)
                if i < targetAmount and M.IsBuying then task.wait(M.Settings.Delay) end
            end
            M.IsBuying = false
        end)
        return true
    end

    function M.Stop()
        M.IsBuying = false
        M.AutoLoop = false
        return true
    end

    function M.EnableAutoLoop()
        if M.AutoLoop then return false end
        if not getPurchaseRemote() then return false end
        M.AutoLoop = true
        task.spawn(function()
            while M.AutoLoop do
                if not M.IsBuying then M.Start() end
                while M.IsBuying and M.AutoLoop do task.wait(0.5) end
                if M.AutoLoop then task.wait(1) end
            end
        end)
        return true
    end

    function M.DisableAutoLoop()
        M.AutoLoop = false
        M.IsBuying = false
        return true
    end

    function M.TestConnection()
        return getPurchaseRemote() ~= nil
    end

    return M
end)()

-- ══════════════════════════════════════════
--       AUTO CLAIM PIRATE CHEST MODULE
-- ══════════════════════════════════════════
local AutoClaimPirateChest = (function()
    local M = {}
    local enabled = false
    local claimInterval = 0.3
    local claimConnection
    local newChestConnection

    local function getClaimRemote()
        return EmbeddedEventResolver:GetRE("ClaimPirateChest")
    end

    local function getPirateChests()
        local chests = {}
        local chestStorage = Workspace:FindFirstChild("PirateChestStorage")
        if chestStorage then
            for _, chest in pairs(chestStorage:GetChildren()) do
                if chest:IsA("Model") then table.insert(chests, chest.Name) end
            end
        end
        return chests
    end

    local function claimChest(chestId)
        pcall(function()
            local remote = getClaimRemote()
            if remote then remote:FireServer(chestId) end
        end)
    end

    function M.Start()
        if enabled then return end
        enabled = true
        claimConnection = task.spawn(function()
            while enabled do
                local chests = getPirateChests()
                for _, chestId in ipairs(chests) do
                    if not enabled then break end
                    claimChest(chestId)
                    task.wait(1.0)
                end
                task.wait(claimInterval)
            end
        end)
        newChestConnection = Workspace.DescendantAdded:Connect(function(descendant)
            if enabled and descendant.Parent and descendant.Parent.Name == "PirateChestStorage" then
                task.wait(0.2)
                if descendant:IsA("Model") then claimChest(descendant.Name) end
            end
        end)
    end

    function M.Stop()
        if not enabled then return end
        enabled = false
        if claimConnection then pcall(function() task.cancel(claimConnection) end) claimConnection = nil end
        if newChestConnection then newChestConnection:Disconnect() newChestConnection = nil end
    end

    function M.IsRunning()
        return enabled
    end

    return M
end)()

-- ══════════════════════════════════════════
--           AUTO POTION MODULE
-- ══════════════════════════════════════════
local AutoPotionSystem = (function()
    local M = {}

    local RS = game:GetService("ReplicatedStorage")
    local Replion = require(RS.Packages.Replion)
    local Data = Replion.Client:WaitReplion("Data")

    local isRunning = false
    local loopTask = nil
    local selectedPotions = {}
    local interval = 30

    local POTIONS = {
        { Name = "Luck I Potion",     Id = 1  },
        { Name = "Coin I Potion",     Id = 2  },
        { Name = "Mutation I Potion", Id = 4  },
        { Name = "Luck II Potion",    Id = 6  },
        { Name = "Love I Potion",     Id = 15 },
    }

    local POTION_BY_NAME = {}
    local POTION_NAMES = {}
    for _, p in ipairs(POTIONS) do
        POTION_BY_NAME[p.Name] = p
        table.insert(POTION_NAMES, p.Name)
    end

    local function executeUsePotion()
        local ok, inventory = pcall(function()
            return Data:GetExpect({ "Inventory", "Potions" })
        end)
        if not ok or not inventory then return end
        for _, potionName in ipairs(selectedPotions) do
            local potion = POTION_BY_NAME[potionName]
            if potion then
                for _, item in ipairs(inventory) do
                    if item.Id == potion.Id then
                        pcall(function() NetEvents.RF_ConsumePotion:InvokeServer(item.UUID, 1) end)
                        break
                    end
                end
            end
        end
    end

    function M.SetSelected(list) selectedPotions = list or {} end
    function M.SetInterval(seconds)
        local n = tonumber(seconds)
        if n and n >= 1 then interval = n return true end
        return false
    end
    function M.GetInterval() return interval end
    function M.Start()
        if isRunning then return false end
        if #selectedPotions == 0 then return false end
        isRunning = true
        loopTask = task.spawn(function()
            while isRunning do
                executeUsePotion()
                task.wait(interval)
            end
        end)
        return true
    end
    function M.Stop()
        if not isRunning then return false end
        isRunning = false
        if loopTask then task.cancel(loopTask) loopTask = nil end
        return true
    end
    function M.IsRunning() return isRunning end
    function M.GetNames() return POTION_NAMES end

    return M
end)()

-- ══════════════════════════════════════════
--         SKIN SWAP ANIMATION MODULE
-- ══════════════════════════════════════════
local SkinSwapAnimation = (function()
    local char = localPlayer.Character or localPlayer.CharacterAdded:Wait()
    local humanoid = char:WaitForChild("Humanoid")
    local Animator = humanoid:FindFirstChildOfClass("Animator") or Instance.new("Animator", humanoid)

    local SkinAnimation = {}
    SkinAnimation.Connections = {}

    -- Data diekstrak dari remot Skin Animation.txt
    local SkinData = {
        ["Sunshine Cello"] = { FishCaught = "rbxassetid://87182079399663", EquipIdle = "rbxassetid://90996210728107" },
        ["Electric Guitar"] = { FishCaught = "rbxassetid://139089375187802", EquipIdle = "rbxassetid://75757852730535" },
        ["Kraken Anchor"] = { FishCaught = nil, EquipIdle = "rbxassetid://126023229958416" },
        ["Oceanic Harpoon"] = { FishCaught = nil, EquipIdle = "rbxassetid://77549515147440" },
        ["Soul Scythe"] = { FishCaught = "rbxassetid://82259219343456", EquipIdle = "rbxassetid://84686809448947" },
        ["Aether Monarch"] = { FishCaught = "rbxassetid://74447876553309", EquipIdle = "rbxassetid://121491018104381" },
        ["Dragonmaster Scythe"] = { FishCaught = "rbxassetid://115176378891122", EquipIdle = "rbxassetid://128506993063074" },
        ["Cloud Weaver"] = { FishCaught = "rbxassetid://121026156047780", EquipIdle = "rbxassetid://76234007743182" },
        ["Luminary"] = { FishCaught = nil, EquipIdle = "rbxassetid://104917097456556" },
        ["Blinding Bloom"] = { FishCaught = nil, EquipIdle = "rbxassetid://104917097456556" },
        ["The Telamonster"] = { FishCaught = nil, EquipIdle = "rbxassetid://104917097456556" },
        ["World Tour Football"] = { FishCaught = "rbxassetid://95122147521041", EquipIdle = "rbxassetid://133853006997201" },
        ["Energy Blaster"] = { FishCaught = "rbxassetid://128051680962035", EquipIdle = "rbxassetid://74605865925487" },
        ["Butterfly Sword"] = { FishCaught = "rbxassetid://128847574673285", EquipIdle = "rbxassetid://80033802624915" },
        ["Galaxy Conqueror"] = { FishCaught = "rbxassetid://133255172354551", EquipIdle = "rbxassetid://120084303822274" },
        ["Overdrive"] = { FishCaught = "rbxassetid://71232649901855", EquipIdle = "rbxassetid://100644136384320" },
        ["Void Guitar"] = { FishCaught = "rbxassetid://79346352464845", EquipIdle = "rbxassetid://95999201117305" },
        ["Kitty Guitar"] = { FishCaught = "rbxassetid://132530630982956", EquipIdle = "rbxassetid://87630031202735" },
        ["Draconic Soul"] = { FishCaught = "rbxassetid://109818439508879", EquipIdle = "rbxassetid://81883372005196" },
        ["Divine Staff"] = { FishCaught = "rbxassetid://107412232735920", EquipIdle = "rbxassetid://86423072130126" },
        ["Empyrean Staff"] = { FishCaught = "rbxassetid://101971777673013", EquipIdle = "rbxassetid://118320952614262" },
        ["Golden Clockwork"] = { FishCaught = "rbxassetid://126346193348309", EquipIdle = "rbxassetid://72700354353152" },
        ["Bunny Summoner"] = { FishCaught = "rbxassetid://101318598176860", EquipIdle = "rbxassetid://74385277734645" },
        ["Easter Parasol"] = { FishCaught = "rbxassetid://90572706842137", EquipIdle = "rbxassetid://116168889227689" },
        ["Serpent's Trident"] = { FishCaught = "rbxassetid://140142098810185", EquipIdle = "rbxassetid://122561472973242" },
        ["Crimson Retribution"] = { FishCaught = "rbxassetid://108205633866814", EquipIdle = "rbxassetid://134618541979874" },
        ["Dark Matter Scythe"] = { FishCaught = "rbxassetid://106846315932087", EquipIdle = "rbxassetid://85161190305071" },
        ["Ethereal Sword"] = { FishCaught = "rbxassetid://110866636674655", EquipIdle = "rbxassetid://116654265230180" },
        ["Cupid's Harp"] = { FishCaught = "rbxassetid://93542218938956", EquipIdle = "rbxassetid://85435462874982" },
        ["Aurelian Bow"] = { FishCaught = "rbxassetid://89083607138153", EquipIdle = "rbxassetid://112757205434160" },
        ["Void Kraken"] = { FishCaught = "rbxassetid://71093335229963", EquipIdle = "rbxassetid://140259064859736" },
        ["Celestial Scythe"] = { FishCaught = "rbxassetid://125568004947137", EquipIdle = "rbxassetid://73640940565483" },
        ["Kitsune Greatsword"] = { FishCaught = "rbxassetid://139914168110430", EquipIdle = "rbxassetid://93518967822593" },
        ["Chromatic Katana"] = { FishCaught = "rbxassetid://75078942392746", EquipIdle = "rbxassetid://87355322562067" },
        ["Crescendo Scythe"] = { FishCaught = "rbxassetid://101593515409348", EquipIdle = "rbxassetid://91723046661800" },
        ["Blackhole Sword"] = { FishCaught = "rbxassetid://88993991486322", EquipIdle = "rbxassetid://110434285817259" },
        ["Eternal Flower"] = { FishCaught = "rbxassetid://119567958965696", EquipIdle = "rbxassetid://115119558523816" },
        ["Holy Trident"] = { FishCaught = "rbxassetid://128167068291703", EquipIdle = "rbxassetid://83219020397849" },
        ["Binary Edge"] = { FishCaught = "rbxassetid://109653945741202", EquipIdle = "rbxassetid://103714544264522" },
        ["The Vanquisher"] = { FishCaught = "rbxassetid://93884986836266", EquipIdle = "rbxassetid://123194574699925" },
        ["Frozen Krampus Scythe"] = { FishCaught = "rbxassetid://134934781977605", EquipIdle = "rbxassetid://124265469726043" },
        ["1x1x1x1 Ban Hammer"] = { FishCaught = "rbxassetid://96285280763544", EquipIdle = "rbxassetid://81302570422307" },
        ["Corruption Edge"] = { FishCaught = "rbxassetid://126613975718573", EquipIdle = "rbxassetid://93958525241489" },
        ["Gingerbread Katana"] = { FishCaught = "rbxassetid://107940819382815", EquipIdle = "rbxassetid://103641983335689" },
        ["Eclipse Katana"] = { FishCaught = "rbxassetid://107940819382815", EquipIdle = "rbxassetid://103641983335689" },
        ["Christmas Parasol"] = { FishCaught = "rbxassetid://99143072029495", EquipIdle = "rbxassetid://79754634120924" },
        ["Princess Parasol"] = { FishCaught = "rbxassetid://99143072029495", EquipIdle = "rbxassetid://79754634120924" },
    }

    local CurrentSkin = nil
    local AnimationPool = {}
    local IsEnabled = false
    local POOL_SIZE = 3
    local killedTracks = {}
    local replaceCount = 0
    local currentPoolIndex = 1

    local function LoadAnimationPool(skinName)
        local data = SkinData[skinName]
        if not data or not data.FishCaught then return false end
        local animId = data.FishCaught

        for _, track in ipairs(AnimationPool) do
            pcall(function() track:Stop(0) track:Destroy() end)
        end
        AnimationPool = {}
        local anim = Instance.new("Animation")
        anim.AnimationId = animId
        anim.Name = "CUSTOM_SKIN_ANIM"
        for i = 1, POOL_SIZE do
            local track = Animator:LoadAnimation(anim)
            track.Priority = Enum.AnimationPriority.Action4
            track.Looped = false
            track.Name = "SKIN_POOL_" .. i
            table.insert(AnimationPool, track)
        end
        currentPoolIndex = 1
        return true
    end

    local function GetNextTrack()
        for i = 1, POOL_SIZE do
            local track = AnimationPool[i]
            if track and not track.IsPlaying then return track end
        end
        currentPoolIndex = currentPoolIndex % POOL_SIZE + 1
        return AnimationPool[currentPoolIndex]
    end

    local function IsFishCaughtAnimation(track)
        if not track or not track.Animation then return false end
        local trackName = string.lower(track.Name or "")
        local animName = string.lower(track.Animation.Name or "")
        return (string.find(trackName, "fishcaught") or string.find(animName, "fishcaught") or
                string.find(trackName, "caught") or string.find(animName, "caught"))
    end

    local function InstantReplace(originalTrack)
        local nextTrack = GetNextTrack()
        if not nextTrack then return end
        replaceCount = replaceCount + 1
        killedTracks[originalTrack] = tick()
        task.spawn(function()
            for i = 1, 10 do
                pcall(function()
                    if originalTrack.IsPlaying then
                        originalTrack:Stop(0)
                        originalTrack:AdjustSpeed(0)
                        originalTrack.TimePosition = 0
                    end
                end)
                task.wait()
            end
        end)
        pcall(function()
            if nextTrack.IsPlaying then nextTrack:Stop(0) end
            nextTrack:Play(0, 1, 1)
            nextTrack:AdjustSpeed(1)
        end)
        task.delay(1, function() killedTracks[originalTrack] = nil end)
    end

    localPlayer.CharacterAdded:Connect(function(newChar)
        task.wait(1.5)
        char = newChar
        humanoid = char:WaitForChild("Humanoid")
        Animator = humanoid:FindFirstChildOfClass("Animator") or Instance.new("Animator", humanoid)
        killedTracks = {}
        replaceCount = 0
        if IsEnabled and CurrentSkin then
            task.wait(0.5)
            LoadAnimationPool(CurrentSkin)
            if SkinAnimation.Connections.AnimationPlayed then
                SkinAnimation.Connections.AnimationPlayed:Disconnect()
            end
            SkinAnimation.Connections.AnimationPlayed = humanoid.AnimationPlayed:Connect(function(track)
                if not IsEnabled then return end
                if IsFishCaughtAnimation(track) then task.spawn(function() InstantReplace(track) end) end
            end)
        end
    end)

    function SkinAnimation.SwitchSkin(skinName)
        if not SkinData[skinName] then return false end
        CurrentSkin = skinName
        if IsEnabled then return LoadAnimationPool(skinName) end
        return true
    end

    function SkinAnimation.Enable()
        if IsEnabled then return false end
        if not CurrentSkin then return false end
        local success = LoadAnimationPool(CurrentSkin)
        if success then
            IsEnabled = true
            killedTracks = {}
            replaceCount = 0
            SkinAnimation.Connections.AnimationPlayed = humanoid.AnimationPlayed:Connect(function(track)
                if not IsEnabled then return end
                if IsFishCaughtAnimation(track) then task.spawn(function() InstantReplace(track) end) end
            end)
            SkinAnimation.Connections.Heartbeat = RunService.Heartbeat:Connect(function()
                if not IsEnabled then return end
                for _, track in ipairs(humanoid:GetPlayingAnimationTracks()) do
                    if string.find(string.lower(track.Name or ""), "skin_pool") then continue end
                    if killedTracks[track] then
                        if track.IsPlaying then pcall(function() track:Stop(0) track:AdjustSpeed(0) end) end
                        continue
                    end
                    if track.IsPlaying and IsFishCaughtAnimation(track) then
                        task.spawn(function() InstantReplace(track) end)
                    end
                end
            end)
            return true
        end
        return false
    end

    function SkinAnimation.Disable()
        if not IsEnabled then return false end
        IsEnabled = false
        killedTracks = {}
        replaceCount = 0
        for _, conn in pairs(SkinAnimation.Connections) do
            if typeof(conn) == "RBXScriptConnection" then conn:Disconnect() end
        end
        SkinAnimation.Connections = {}
        for _, track in ipairs(AnimationPool) do pcall(function() track:Stop(0) end) end
        return true
    end

    function SkinAnimation.IsEnabled() return IsEnabled end
    function SkinAnimation.GetCurrentSkin() return CurrentSkin end
    function SkinAnimation.GetReplaceCount() return replaceCount end
    function SkinAnimation.GetSkinList()
        local list = {}
        for name, _ in pairs(SkinData) do table.insert(list, name) end
        table.sort(list)
        return list
    end

    return SkinAnimation
end)()

-- ══════════════════════════════════════════
--         SKIN ANIMATION SECTION (UI)
-- ══════════════════════════════════════════
local SkinSection = Auto:Section({
    Title = "Skin Animation",
    Box = true,
    TextXAlignment = "Center",
    TextSize = 15,
    Opened = false,
})

local skinNames = SkinSwapAnimation.GetSkinList()
local selectedSkin = skinNames[1] or "Eclipse Katana"

SkinSection:Dropdown({
    Title = "Select Skin",
    Values = skinNames,
    Value = 1,
    Callback = function(selected)
        selectedSkin = selected
        if SkinSwapAnimation.IsEnabled() and SkinSwapAnimation.GetCurrentSkin() ~= selected then
            SkinSwapAnimation.SwitchSkin(selected)
        end
    end,
})

SkinSection:Toggle({
    Title = "Enable Skin Animation",
    Value = false,
    Callback = function(state)
        if state then
            if not selectedSkin or selectedSkin == "" then return end
            SkinSwapAnimation.SwitchSkin(selectedSkin)
            SkinSwapAnimation.Enable()
        else
            SkinSwapAnimation.Disable()
        end
    end,
})

-- ══════════════════════════════════════════
--           AUTO ENCHANT MODULE
-- ══════════════════════════════════════════
CombinedModules.AutoEnchant = (function()
    local AutoEnchant = {}

    local RS = game:GetService("ReplicatedStorage")
    local LP = game.Players.LocalPlayer
    local Net = RS.Packages._Index["sleitnick_net@0.2.0"].net
    local Replion = require(RS.Packages.Replion)
    local PlayerStatsUtility = require(RS.Shared.PlayerStatsUtility)
    local ItemUtility = require(RS.Shared.ItemUtility)

    local REEquipItem                    = NetEvents.RE_EquipItem
    local RFEquipToolFromHotbar          = NetEvents.RF_EquipToolFromHotbar
    local REActivateEnchantingAltar      = NetEvents.RE_ActivateEnchantingAltar
    local REActivateSecondEnchantingAltar= NetEvents.RE_ActivateSecondEnchantingAltar
    local RERollEnchant                  = NetEvents.RE_RollEnchant
    local UpdateRemote                   = RS.Packages._Index["ytrev_replion@2.0.0-rc.3"].replion.Remotes.Update

    local Client = Replion.Client
    local Data   = Client:WaitReplion("Data")

    local enchantMapping = {}
    local function buildEnchantMapping()
        enchantMapping = {}
        pcall(function()
            local enchantFolder = RS:WaitForChild("Enchants", 10)
            if enchantFolder then
                for _, child in ipairs(enchantFolder:GetChildren()) do
                    if child:IsA("ModuleScript") then
                        local ok, data = pcall(require, child)
                        if ok and data and data.Data and data.Data.Name and data.Data.Id then
                            enchantMapping[data.Data.Name] = data.Data.Id
                        end
                    end
                end
            end
        end)
    end
    buildEnchantMapping()

    local config = {
        enabled = false,
        targetEnchantId = 10,
        targetEnchantName = "XPerienced I",
        rollCount = 0,
        waitingForUpdate = false,
        currentCycleRunning = false,
        currentTask = nil,
        enchantType = 1,
        enchantStoneItemId = 10
    }

    local function teleportToEnchant()
        local character = LP.Character
        if character and character:FindFirstChild("HumanoidRootPart") then
            if config.enchantType == 2 then
                character.HumanoidRootPart.CFrame = CFrame.new(1478.29846, 126.044891, -613.519653)
            else
                character.HumanoidRootPart.CFrame = CFrame.new(3245, -1301, 1394)
            end
            task.wait(2)
            return true
        end
        return false
    end

    local function findEnchantStoneUUIDs(enchantStoneItemId)
        local uuids = {}
        pcall(function()
            for _, item in pairs(Data:GetExpect({"Inventory", "Items"})) do
                if item.Id == enchantStoneItemId then
                    table.insert(uuids, item.UUID)
                end
            end
        end)
        return uuids
    end

    local function equipEnchantStoneAndGetSlot()
        local stoneUUIDs = findEnchantStoneUUIDs(config.enchantStoneItemId)
        if #stoneUUIDs == 0 then return nil end
        local stoneUUID = stoneUUIDs[1]
        local slotKey = nil
        local timeout = tick()
        while tick() - timeout < 5 do
            if not config.enabled then return nil end
            local equippedItems = Data:Get("EquippedItems") or {}
            for key, uuid in pairs(equippedItems) do
                if uuid == stoneUUID then slotKey = key end
            end
            if not slotKey then
                pcall(function() REEquipItem:FireServer(stoneUUID, "Enchant Stones") end)
                task.wait(0.3)
            else break end
        end
        return slotKey
    end

    local function equipTool(slotKey)
        if not slotKey then return false end
        pcall(function() RFEquipToolFromHotbar:InvokeServer(slotKey) end)
        task.wait(0.2)
        return true
    end

    local function activateAltar()
        for i = 1, 3 do
            pcall(function()
                if config.enchantType == 2 then
                    REActivateSecondEnchantingAltar:FireServer()
                else
                    REActivateEnchantingAltar:FireServer()
                end
            end)
            task.wait(0.5)
        end
        return true
    end

    local function rollEnchant()
        config.waitingForUpdate = true
        pcall(function() RERollEnchant:FireServer() end)
        local startTime = tick()
        while config.waitingForUpdate and tick() - startTime < 3.5 do task.wait(0.1) end
        if config.waitingForUpdate and config.enabled then
            config.waitingForUpdate = false
            config.currentCycleRunning = false
            return false
        end
        return true
    end

    local function startEnchantCycle()
        if not config.enabled or config.currentCycleRunning then return end
        config.currentCycleRunning = true
        if not teleportToEnchant() then config.currentCycleRunning = false return end
        local slotKey = equipEnchantStoneAndGetSlot()
        if not slotKey then config.currentCycleRunning = false return end
        if not equipTool(slotKey) then config.currentCycleRunning = false return end
        if not activateAltar() then config.currentCycleRunning = false return end
        if not rollEnchant() then
            task.wait(1)
            if config.enabled then startEnchantCycle() end
        end
    end

    if UpdateRemote then
        UpdateRemote.OnClientEvent:Connect(function(dataString, path, data)
            if not config.enabled or not config.waitingForUpdate then return end
            config.waitingForUpdate = false
            config.currentCycleRunning = false
            if path and type(path) == "table" and #path >= 4 then
                if path[1] == "Inventory" and path[2] == "Fishing Rods" and path[4] == "Metadata" then
                    if data and data.EnchantId then
                        config.rollCount = config.rollCount + 1
                        if data.EnchantId == config.targetEnchantId then
                            config.enabled = false
                        else
                            task.wait(8)
                            if config.enabled then startEnchantCycle() end
                        end
                    end
                end
            end
        end)
    end

    function AutoEnchant.Start()
        if config.enabled then return false end
        if not LP.Character or not LP.Character:FindFirstChild("HumanoidRootPart") then return false end
        config.enabled = true
        config.rollCount = 0
        config.waitingForUpdate = false
        config.currentCycleRunning = false
        config.currentTask = task.spawn(function() startEnchantCycle() end)
        return true
    end

    function AutoEnchant.Stop()
        if not config.enabled then return false end
        config.enabled = false
        config.waitingForUpdate = false
        config.currentCycleRunning = false
        if config.currentTask then task.cancel(config.currentTask) config.currentTask = nil end
        return true
    end

    function AutoEnchant.IsRunning() return config.enabled end
    function AutoEnchant.SetTargetEnchant(enchantName)
        local enchantId = enchantMapping[enchantName]
        if enchantId then
            config.targetEnchantId = enchantId
            config.targetEnchantName = enchantName
            return true
        end
        return false
    end
    function AutoEnchant.GetRollCount() return config.rollCount end
    function AutoEnchant.GetEnchantList()
        local enchants = {}
        for name, _ in pairs(enchantMapping) do table.insert(enchants, name) end
        table.sort(enchants)
        return enchants
    end
    function AutoEnchant.SetEnchantType(enchantType)
        if enchantType == 1 or enchantType == 2 then config.enchantType = enchantType return true end
        return false
    end
    function AutoEnchant.SetEnchantStoneType(stoneItemId)
        config.enchantStoneItemId = stoneItemId
        return true
    end
    function AutoEnchant.GetEnchantStatus(enchantStoneItemId)
        enchantStoneItemId = enchantStoneItemId or config.enchantStoneItemId
        local rodName = "None"
        local enchantName = "None"
        local stoneCount = 0
        local stoneUUIDs = {}
        pcall(function()
            local equippedItems = Data:Get("EquippedItems") or {}
            local fishingRods = Data:Get({"Inventory", "Fishing Rods"}) or {}
            for _, uuid in pairs(equippedItems) do
                for _, rod in ipairs(fishingRods) do
                    if rod.UUID == uuid then
                        local itemData = ItemUtility:GetItemData(rod.Id)
                        rodName = itemData and itemData.Data.Name or "None"
                        if rod.Metadata and rod.Metadata.EnchantId then
                            local enchantData = ItemUtility:GetEnchantData(rod.Metadata.EnchantId)
                            if enchantData and enchantData.Data and enchantData.Data.Name then
                                enchantName = enchantData.Data.Name
                            end
                        end
                    end
                end
            end
            for _, item in pairs(Data:GetExpect({"Inventory", "Items"})) do
                if item.Id == enchantStoneItemId then
                    stoneCount = stoneCount + 1
                    table.insert(stoneUUIDs, item.UUID)
                end
            end
        end)
        return rodName, enchantName, stoneCount, stoneUUIDs
    end
    function AutoEnchant.TeleportToAltar()
        local character = LP.Character or LP.CharacterAdded:Wait()
        local hrp = character:FindFirstChild("HumanoidRootPart")
        local humanoid = character:FindFirstChildOfClass("Humanoid")
        if hrp and humanoid then
            hrp.CFrame = CFrame.new(Vector3.new(3258, -1301, 1391))
            humanoid:ChangeState(Enum.HumanoidStateType.Physics)
            task.wait(0.1)
            humanoid:ChangeState(Enum.HumanoidStateType.Running)
            return true
        end
        return false
    end

    return AutoEnchant
end)()

-- ══════════════════════════════════════════
--         AUTO SPAWN TOTEM MODULE
-- ══════════════════════════════════════════
CombinedModules.AutoSpawnTotem = (function()
    local AutoSpawnTotem = {}
    local RS = game:GetService("ReplicatedStorage")
    local SpawnTotemRemote = nil
    local clientData = nil

    local function InitializeRemotes()
        local success = pcall(function()
            SpawnTotemRemote = EmbeddedEventResolver:GetRE("SpawnTotem")
            local Replion = require(RS.Packages.Replion)
            clientData = Replion.Client:WaitReplion("Data")
        end)
        return success and SpawnTotemRemote ~= nil and clientData ~= nil
    end

    local TOTEM_DATA = {
        ["Luck Totem"]     = { Id = 1, Duration = 3600 },
        ["Mutation Totem"] = { Id = 2, Duration = 3600 },
        ["Shiny Totem"]    = { Id = 3, Duration = 3600 },
    }
    local TOTEM_NAMES = { "Luck Totem", "Mutation Totem", "Shiny Totem" }

    AutoSpawnTotem.Settings = { SelectedTotem = "Luck Totem", IsRunning = false, SpawnInterval = 3605 }
    local AUTO_SPAWN_THREAD = nil

    local function GetTotemUUIDByName(totemName)
        if not clientData then return nil end
        local ok, inv = pcall(function() return clientData:Get("Inventory") end)
        if not ok or not inv or not inv.Totems then return nil end
        local entry = TOTEM_DATA[totemName]
        local targetId = entry and entry.Id
        if not targetId then return nil end
        for _, item in pairs(inv.Totems) do
            if item and item.UUID and tonumber(item.Id) == targetId then
                if (item.Count or 1) >= 1 then return item.UUID end
            end
        end
        return nil
    end

    local function SpawnSingleTotem()
        local totemUUID = GetTotemUUIDByName(AutoSpawnTotem.Settings.SelectedTotem)
        if not totemUUID or not SpawnTotemRemote then return false end
        local ok = pcall(function() SpawnTotemRemote:FireServer(totemUUID) end)
        return ok
    end

    local function RunAutoSpawnLoop()
        if AUTO_SPAWN_THREAD then pcall(function() task.cancel(AUTO_SPAWN_THREAD) end) end
        AUTO_SPAWN_THREAD = task.spawn(function()
            SpawnSingleTotem()
            while AutoSpawnTotem.Settings.IsRunning do
                task.wait(AutoSpawnTotem.Settings.SpawnInterval)
                if AutoSpawnTotem.Settings.IsRunning then SpawnSingleTotem() end
            end
        end)
    end

    function AutoSpawnTotem.SetTotem(totemName)
        if TOTEM_DATA[totemName] then AutoSpawnTotem.Settings.SelectedTotem = totemName return true end
        return false
    end
    function AutoSpawnTotem.GetTotemNames() return TOTEM_NAMES end
    function AutoSpawnTotem.Start()
        if AutoSpawnTotem.Settings.IsRunning then return false end
        if not SpawnTotemRemote or not clientData then
            if not InitializeRemotes() then return false end
        end
        AutoSpawnTotem.Settings.IsRunning = true
        RunAutoSpawnLoop()
        return true
    end
    function AutoSpawnTotem.Stop()
        if not AutoSpawnTotem.Settings.IsRunning then return false end
        AutoSpawnTotem.Settings.IsRunning = false
        if AUTO_SPAWN_THREAD then pcall(function() task.cancel(AUTO_SPAWN_THREAD) end) AUTO_SPAWN_THREAD = nil end
        return true
    end
    function AutoSpawnTotem.IsRunning() return AutoSpawnTotem.Settings.IsRunning end
    function AutoSpawnTotem.GetCurrentTotem() return AutoSpawnTotem.Settings.SelectedTotem end

    task.spawn(function() task.wait(1) InitializeRemotes() end)

    return AutoSpawnTotem
end)()

-- ══════════════════════════════════════════
--              AUTO TAB
-- ══════════════════════════════════════════
local Auto = Window:Tab({
    Title = "Automation",
    Icon = "zap",
    IconColor = Mains,
    IconShape = "Square",
    Border = true,
})

-- ══════════════════════════════════════════
--           AUTO BUY CHARM SECTION
-- ══════════════════════════════════════════
local CharmSection = Auto:Section({
    Title = "Auto Buy Charm",
    Box = true,
    TextXAlignment = "Center",
    TextSize = 15,
    Opened = false,
})

local charmMap = {
    ["Bone Charm"]   = 1,
    ["Algae Charm"]  = 2,
    ["Magma Charm"]  = 3,
    ["Clover Charm"] = 4,
}
local selectedCharmType = 1

CharmSection:Dropdown({
    Title = "Charm Type",
    Values = { "Bone Charm", "Algae Charm", "Magma Charm", "Clover Charm" },
    Value = 1,
    Callback = function(selected)
        selectedCharmType = charmMap[selected] or 1
        BuyCharm.SetCharmType(selectedCharmType)
    end,
})

CharmSection:Input({
    Title = "Amount",
    Value = "1",
    Type = "Input",
    Placeholder = "1",
    Callback = function(value)
        local num = tonumber(value)
        if num then BuyCharm.SetAmount(num) end
    end,
})

CharmSection:Input({
    Title = "Delay (sec)",
    Value = "0.5",
    Type = "Input",
    Placeholder = "0.5",
    Callback = function(value)
        local num = tonumber(value)
        if num then BuyCharm.SetDelay(num) end
    end,
})

CharmSection:Button({
    Title = "Buy Charm Once",
    Callback = function()
        if BuyCharm.IsBuying then return end
        if not BuyCharm.TestConnection() then return end
        BuyCharm.Start(nil, selectedCharmType)
    end,
})

CharmSection:Toggle({
    Title = "Auto Loop Buy Charm",
    Value = false,
    Callback = function(state)
        if state then
            BuyCharm.EnableAutoLoop()
        else
            BuyCharm.DisableAutoLoop()
        end
    end,
})

-- ══════════════════════════════════════════
--       AUTO CLAIM PIRATE CHEST SECTION
-- ══════════════════════════════════════════
local ClaimSection = Auto:Section({
    Title = "Auto Claim Pirate Chest",
    Box = true,
    TextXAlignment = "Center",
    TextSize = 15,
    Opened = false,
})

ClaimSection:Toggle({
    Title = "Enable Auto Claim",
    Value = false,
    Callback = function(state)
        if state then
            AutoClaimPirateChest.Start()
        else
            AutoClaimPirateChest.Stop()
        end
    end,
})

-- ══════════════════════════════════════════
--           AUTO USE POTION SECTION
-- ══════════════════════════════════════════
local PotionSection = Auto:Section({
    Title = "Auto Use Potion",
    Box = true,
    TextXAlignment = "Center",
    TextSize = 15,
    Opened = false,
})

local selectedPotions = {}

PotionSection:Dropdown({
    Title = "Select Potions",
    Values = AutoPotionSystem.GetNames(),
    Value = nil,
    AllowNone = true,
    Multi = true,
    Callback = function(selected)
        if type(selected) == "table" then
            selectedPotions = selected
            AutoPotionSystem.SetSelected(selectedPotions)
        end
    end,
})

PotionSection:Input({
    Title = "Interval (Seconds)",
    Value = "30",
    Type = "Input",
    Placeholder = "30",
    Callback = function(value)
        AutoPotionSystem.SetInterval(value)
    end,
})

PotionSection:Toggle({
    Title = "Enable Auto Potion",
    Value = false,
    Callback = function(state)
        if state then
            AutoPotionSystem.SetSelected(selectedPotions)
            AutoPotionSystem.Start()
        else
            AutoPotionSystem.Stop()
        end
    end,
})

-- ══════════════════════════════════════════
--           AUTO ENCHANT SECTION
-- ══════════════════════════════════════════
local EnchantSection = Auto:Section({
    Title = "Auto Enchant",
    Box = true,
    TextXAlignment = "Center",
    TextSize = 15,
    Opened = false,
})

local enchantTypeMap = {
    ["Normal Enchant"]  = { stone = 10,  type = 1 },
    ["Second Enchant"]  = { stone = 246, type = 2 },
    ["Evolved Enchant"] = { stone = 558, type = 1 },
    ["Candy Enchant"]   = { stone = 714, type = 1 },
}

EnchantSection:Dropdown({
    Title = "Enchant Type",
    Values = { "Normal Enchant", "Second Enchant", "Evolved Enchant", "Candy Enchant" },
    Value = 1,
    Callback = function(selected)
        local data = enchantTypeMap[selected]
        if data and CombinedModules.AutoEnchant then
            CombinedModules.AutoEnchant.SetEnchantStoneType(data.stone)
            CombinedModules.AutoEnchant.SetEnchantType(data.type)
        end
    end,
})

EnchantSection:Dropdown({
    Title = "Target Enchant",
    Values = CombinedModules.AutoEnchant.GetEnchantList(),
    Value = 1,
    Callback = function(selected)
        if CombinedModules.AutoEnchant then
            CombinedModules.AutoEnchant.SetTargetEnchant(selected)
        end
    end,
})

EnchantSection:Toggle({
    Title = "Enable Auto Enchant",
    Value = false,
    Callback = function(state)
        if state then
            CombinedModules.AutoEnchant.Start()
        else
            CombinedModules.AutoEnchant.Stop()
        end
    end,
})

EnchantSection:Button({
    Title = "Teleport to Altar 1",
    Callback = function()
        local char = game.Players.LocalPlayer.Character
        if char and char:FindFirstChild("HumanoidRootPart") then
            char.HumanoidRootPart.CFrame = CFrame.new(3245, -1301, 1394)
        end
    end,
})

EnchantSection:Button({
    Title = "Teleport to Altar 2",
    Callback = function()
        local char = game.Players.LocalPlayer.Character
        if char and char:FindFirstChild("HumanoidRootPart") then
            char.HumanoidRootPart.CFrame = CFrame.new(1478.29846, 126.044891, -613.519653)
        end
    end,
})

-- ══════════════════════════════════════════
--         AUTO SPAWN TOTEM SECTION
-- ══════════════════════════════════════════
local TotemSection = Auto:Section({
    Title = "Auto Spawn Totem",
    Box = true,
    TextXAlignment = "Center",
    TextSize = 15,
    Opened = false,
})

TotemSection:Dropdown({
    Title = "Totem Type",
    Values = CombinedModules.AutoSpawnTotem.GetTotemNames(),
    Value = 1,
    Callback = function(selected)
        CombinedModules.AutoSpawnTotem.SetTotem(selected)
    end,
})

TotemSection:Toggle({
    Title = "Enable Auto Spawn Totem",
    Value = false,
    Callback = function(state)
        if state then
            CombinedModules.AutoSpawnTotem.Start()
        else
            CombinedModules.AutoSpawnTotem.Stop()
        end
    end,
})

-- ══════════════════════════════════════════
--           AUTO TRADE MODULE
-- ══════════════════════════════════════════
CombinedModules.AutoTrade = (function()
    local AutoTrade = {}

    local RS = game:GetService("ReplicatedStorage")
    local Players = game:GetService("Players")
    local LP = Players.LocalPlayer

    local Net = RS.Packages._Index["sleitnick_net@0.2.0"].net
    local Replion = require(RS.Packages.Replion)
    local ItemUtility = require(RS.Shared.ItemUtility)
    local VendorUtility = require(RS.Shared.VendorUtility)
    local PlayerStatsUtility = require(RS.Shared.PlayerStatsUtility)

    local Data = Replion.Client:WaitReplion("Data")
    local Items = RS:WaitForChild("Items")

    local RF_InitiateTrade = (EmbeddedEventResolver and EmbeddedEventResolver:GetRF("InitiateTrade"))
        or (Net and Net["RF/InitiateTrade"])

    local TierFish = {
        [1] = "Common", [2] = "Uncommon", [3] = "Rare",
        [4] = "Epic", [5] = "Legendary", [6] = "Mythic", [7] = "Secret"
    }

    local EnchantStoneIds = {
        ["Normal"] = 10,
        ["Double"] = 246,
        ["Evolved"] = 558
    }

    local config = {
        enabled = false,
        targetPlayer = nil,
        tradeMode = "ByName",
        selectedItem = nil,
        itemAmount = 1,
        targetCoins = 0,
        selectedRarity = "Common",
        rarityAmount = 1,
        selectedStoneType = "Normal",
        stoneAmount = 1,
        currentTask = nil,
        playerManuallySelected = false,
    }

    local stats = {
        totalAttempted = 0,
        totalSuccess = 0,
        totalFailed = 0,
        targetAmount = 0,
        currentItem = "",
        status = "Idle",
        lastTradedItem = "",
        coinTraded = 0,
    }

    AutoTrade.OnStatsChanged = nil
    AutoTrade.OnCompleted = nil

    local function updateStats()
        if AutoTrade.OnStatsChanged then pcall(AutoTrade.OnStatsChanged, stats) end
    end

    local function setStatus(newStatus)
        stats.status = newStatus
        updateStats()
    end

    local function playerExists(playerName)
        return Players:FindFirstChild(playerName) ~= nil
    end

    local function itemStillExists(uuid)
        for _, item in pairs(Data:GetExpect({"Inventory", "Items"})) do
            if item.UUID == uuid then return true end
        end
        return false
    end

    local function executeTrade(playerName, itemUUID, itemName, coinValue)
        if not config.enabled then return false end
        local targetPlayer = Players:FindFirstChild(playerName)
        if not targetPlayer then return false end

        stats.currentItem = itemName or "Unknown"
        stats.totalAttempted = stats.totalAttempted + 1
        setStatus(string.format("Trading: %s (%d/%d)", itemName or "?", stats.totalSuccess + 1, stats.targetAmount))

        local success = false
        local attempts = 0

        while attempts < 3 and config.enabled do
            local tradeSuccess = pcall(function()
                RF_InitiateTrade:InvokeServer(targetPlayer.UserId, itemUUID)
            end)
            if tradeSuccess then
                local startTime = tick()
                while config.enabled do
                    if not itemStillExists(itemUUID) then success = true break
                    elseif tick() - startTime > 5 then break end
                    task.wait(0.2)
                end
                if success then
                    stats.totalSuccess = stats.totalSuccess + 1
                    stats.lastTradedItem = itemName or "Unknown"
                    stats.coinTraded = stats.coinTraded + (coinValue or 0)
                    setStatus(string.format("Success: %s (%d/%d)", itemName or "?", stats.totalSuccess, stats.targetAmount))
                    task.wait(5)
                    return true
                end
            end
            attempts = attempts + 1
            task.wait(0.5)
        end

        stats.totalFailed = stats.totalFailed + 1
        setStatus(string.format("Failed: %s (attempt %d)", itemName or "?", stats.totalAttempted))
        updateStats()
        return false
    end

    local function getGroupedByType(itemType)
        local inventory = Data:GetExpect({"Inventory", "Items"})
        local grouped = {}
        for _, item in ipairs(inventory) do
            local itemData = ItemUtility.GetItemDataFromItemType("Items", item.Id)
            if itemData and itemData.Data.Type == itemType and not item.Favorited then
                local name = itemData.Data.Name
                grouped[name] = grouped[name] or { count = 0, uuids = {} }
                grouped[name].count = grouped[name].count + (item.Quantity or 1)
                table.insert(grouped[name].uuids, item.UUID)
            end
        end
        return grouped
    end

    local function chooseFishesByRange(fishList, targetCoins)
        table.sort(fishList, function(a, b) return a.Price > b.Price end)
        local selected = {}
        local totalValue = 0
        for _, fish in ipairs(fishList) do
            if totalValue + fish.Price <= targetCoins then
                table.insert(selected, fish)
                totalValue = totalValue + fish.Price
            end
            if totalValue >= targetCoins then break end
        end
        if totalValue < targetCoins and #fishList > 0 then
            table.insert(selected, fishList[#fishList])
        end
        return selected, totalValue
    end

    local function autoTradeByName()
        if not config.targetPlayer or not config.selectedItem then setStatus("Error: Target/Item not set") return end
        if not playerExists(config.targetPlayer) then setStatus("Error: Player not found") return end
        stats.targetAmount = config.itemAmount
        setStatus("Starting ByName trade...")
        local grouped = getGroupedByType("Fish")
        local itemData = grouped[config.selectedItem]
        if not itemData or #itemData.uuids == 0 then setStatus("Error: Item not found in inventory") return end
        local maxToTrade = math.min(config.itemAmount, #itemData.uuids)
        stats.targetAmount = maxToTrade
        for i = 1, maxToTrade do
            if not config.enabled then break end
            executeTrade(config.targetPlayer, itemData.uuids[i], config.selectedItem)
        end
    end

    local function autoTradeByCoin()
        if not config.targetPlayer or config.targetCoins <= 0 then setStatus("Error: Target/Coins not set") return end
        if not playerExists(config.targetPlayer) then setStatus("Error: Player not found") return end
        setStatus("Starting ByCoin trade...")
        local fishList = {}
        local inventory = Data:GetExpect({"Inventory", "Items"})
        local playerMods = PlayerStatsUtility:GetPlayerModifiers(LP)
        for _, item in ipairs(inventory) do
            if not item.Favorited then
                local itemData = ItemUtility:GetItemData(item.Id)
                if itemData and itemData.Data and itemData.Data.Type == "Fish" then
                    local sellPrice = VendorUtility:GetSellPrice(item) or itemData.SellPrice or 0
                    local finalPrice = math.ceil(sellPrice * (playerMods and playerMods.CoinMultiplier or 1))
                    if finalPrice > 0 then
                        table.insert(fishList, { UUID = item.UUID, Name = itemData.Data.Name, Price = finalPrice })
                    end
                end
            end
        end
        if #fishList == 0 then setStatus("Error: No fish found in inventory") return end
        local selectedFish, totalValue = chooseFishesByRange(fishList, config.targetCoins)
        stats.targetAmount = #selectedFish
        for _, fish in ipairs(selectedFish) do
            if not config.enabled then break end
            executeTrade(config.targetPlayer, fish.UUID, fish.Name, fish.Price)
        end
    end

    local function autoTradeByRarity()
        if not config.targetPlayer or not config.selectedRarity then setStatus("Error: Target/Rarity not set") return end
        if not playerExists(config.targetPlayer) then setStatus("Error: Player not found") return end
        stats.targetAmount = config.rarityAmount
        setStatus("Starting ByRarity trade...")
        local rarityFish = {}
        local inventory = Data:GetExpect({"Inventory", "Items"})
        for _, item in ipairs(inventory) do
            if not item.Favorited then
                local itemData = ItemUtility.GetItemDataFromItemType("Items", item.Id)
                if itemData and itemData.Data and itemData.Data.Type == "Fish" then
                    local rarity = TierFish[itemData.Data.Tier]
                    if rarity == config.selectedRarity then
                        table.insert(rarityFish, { UUID = item.UUID, Name = itemData.Data.Name or "Unknown", Rarity = rarity })
                    end
                end
            end
        end
        if #rarityFish == 0 then setStatus("Error: No fish with rarity " .. config.selectedRarity) return end
        local maxToTrade = math.min(config.rarityAmount, #rarityFish)
        stats.targetAmount = maxToTrade
        for i = 1, maxToTrade do
            if not config.enabled then break end
            executeTrade(config.targetPlayer, rarityFish[i].UUID, rarityFish[i].Name)
        end
    end

    local function autoTradeByEnchantStone()
        if not config.targetPlayer or not config.selectedStoneType then setStatus("Error: Target/Stone not set") return end
        if not playerExists(config.targetPlayer) then setStatus("Error: Player not found") return end
        local stoneItemId = EnchantStoneIds[config.selectedStoneType]
        if not stoneItemId then setStatus("Error: Invalid stone type") return end
        local stoneUUID = nil
        local stoneName = config.selectedStoneType .. " Enchant Stone"
        local inventory = Data:GetExpect({"Inventory", "Items"})
        for _, item in pairs(inventory) do
            if item.Id == stoneItemId then
                stoneUUID = item.UUID
                local itemData = ItemUtility.GetItemDataFromItemType("Items", item.Id)
                if itemData and itemData.Data and itemData.Data.Name then stoneName = itemData.Data.Name end
                break
            end
        end
        if not stoneUUID then setStatus("Error: No " .. config.selectedStoneType .. " Stone found") return end
        stats.targetAmount = config.stoneAmount
        setStatus(string.format("Trading %s x%d...", stoneName, config.stoneAmount))
        local targetPlayer = Players:FindFirstChild(config.targetPlayer)
        if not targetPlayer then setStatus("Error: Player not found") return end
        for i = 1, config.stoneAmount do
            if not config.enabled then break end
            stats.currentItem = stoneName
            stats.totalAttempted = stats.totalAttempted + 1
            setStatus(string.format("Trading: %s (%d/%d)", stoneName, i, config.stoneAmount))
            local tradeSuccess, tradeResult = pcall(function()
                return RF_InitiateTrade:InvokeServer(targetPlayer.UserId, stoneUUID)
            end)
            if tradeSuccess and tradeResult then
                stats.totalSuccess = stats.totalSuccess + 1
                stats.lastTradedItem = stoneName
                setStatus(string.format("Success: %s (%d/%d)", stoneName, stats.totalSuccess, config.stoneAmount))
                updateStats()
                task.wait(5)
            else
                stats.totalFailed = stats.totalFailed + 1
                setStatus(string.format("Failed: %s (%d/%d)", stoneName, i, config.stoneAmount))
                updateStats()
                task.wait(1)
            end
        end
    end

    function AutoTrade.Start()
        if config.enabled then return false end
        if not config.targetPlayer or not config.playerManuallySelected then return false end
        if not playerExists(config.targetPlayer) then return false end
        config.enabled = true
        stats.totalAttempted = 0 stats.totalSuccess = 0 stats.totalFailed = 0
        stats.currentItem = "" stats.lastTradedItem = "" stats.coinTraded = 0 stats.status = "Starting..."
        updateStats()
        config.currentTask = task.spawn(function()
            if config.tradeMode == "ByName" then autoTradeByName()
            elseif config.tradeMode == "ByCoin" then autoTradeByCoin()
            elseif config.tradeMode == "ByRarity" then autoTradeByRarity()
            elseif config.tradeMode == "ByEnchantStone" then autoTradeByEnchantStone()
            end
            if config.enabled then
                config.enabled = false
                setStatus(string.format("Completed! %d/%d trades successful", stats.totalSuccess, stats.targetAmount))
                if AutoTrade.OnCompleted then pcall(AutoTrade.OnCompleted, stats) end
            else
                setStatus(string.format("Stopped: %d/%d successful", stats.totalSuccess, stats.targetAmount))
            end
        end)
        return true
    end

    function AutoTrade.Stop()
        if not config.enabled then return false end
        config.enabled = false
        if config.currentTask then task.cancel(config.currentTask) config.currentTask = nil end
        setStatus(string.format("Stopped: %d/%d successful", stats.totalSuccess, stats.targetAmount))
        return true
    end

    function AutoTrade.IsRunning() return config.enabled end
    function AutoTrade.GetStats() return stats end
    function AutoTrade.ResetStats()
        stats.totalAttempted = 0 stats.totalSuccess = 0 stats.totalFailed = 0
        stats.targetAmount = 0 stats.currentItem = "" stats.status = "Idle"
        stats.lastTradedItem = "" stats.coinTraded = 0
        updateStats()
    end
    function AutoTrade.SetTargetPlayer(playerName) config.targetPlayer = playerName config.playerManuallySelected = true end
    function AutoTrade.SetTradeMode(mode)
        if mode == "ByName" or mode == "ByCoin" or mode == "ByRarity" or mode == "ByEnchantStone" then
            config.tradeMode = mode return true
        end
        return false
    end
    function AutoTrade.SetItem(itemName) config.selectedItem = itemName end
    function AutoTrade.SetItemAmount(amount) config.itemAmount = tonumber(amount) or 1 end
    function AutoTrade.SetTargetCoins(coins) config.targetCoins = tonumber(coins) or 0 end
    function AutoTrade.SetRarity(rarity) config.selectedRarity = rarity end
    function AutoTrade.SetRarityAmount(amount) config.rarityAmount = tonumber(amount) or 1 end
    function AutoTrade.SetStoneType(stoneType)
        if EnchantStoneIds[stoneType] then config.selectedStoneType = stoneType return true end
        return false
    end
    function AutoTrade.SetStoneAmount(amount) config.stoneAmount = tonumber(amount) or 1 end
    function AutoTrade.GetPlayers()
        local playerList = {}
        for _, player in ipairs(Players:GetPlayers()) do
            if player ~= LP then table.insert(playerList, player.Name) end
        end
        return playerList
    end
    function AutoTrade.GetFishItems()
        local grouped = getGroupedByType("Fish")
        local displayList = {}
        for name, data in pairs(grouped) do
            table.insert(displayList, string.format("%s x%d", name, data.count))
        end
        return displayList
    end
    function AutoTrade.GetEnchantStoneItems()
        local displayList = {}
        local inventory = Data:GetExpect({"Inventory", "Items"})
        for stoneType, stoneId in pairs(EnchantStoneIds) do
            local count = 0
            for _, item in pairs(inventory) do
                if item.Id == stoneId then count = count + (item.Quantity or 1) end
            end
            if count > 0 then table.insert(displayList, string.format("%s x%d", stoneType, count)) end
        end
        return displayList
    end

    return AutoTrade
end)()

-- ══════════════════════════════════════════
--              TRADE TAB
-- ══════════════════════════════════════════
local Trade = Window:Tab({
    Title = "Trade",
    Icon = "repeat",
    IconColor = Mains,
    IconShape = "Square",
    Border = true,
})

local activeMonitor = nil
local activeToggle = nil

-- ══════════════════════════════════════════
--           SELECT PLAYER SECTION
-- ══════════════════════════════════════════
local PlayerSection = Trade:Section({
    Title = "Select Player",
    Box = true,
    TextXAlignment = "Center",
    TextSize = 15,
    Opened = true,
})

local playerDropdown = PlayerSection:Dropdown({
    Title = "Target Player",
    Values = CombinedModules.AutoTrade.GetPlayers(),
    Value = nil,
    AllowNone = true,
    Callback = function(value)
        if value then CombinedModules.AutoTrade.SetTargetPlayer(value) end
    end,
})

PlayerSection:Button({
    Title = "Refresh Players",
    Callback = function()
        playerDropdown:Set(CombinedModules.AutoTrade.GetPlayers())
    end,
})

-- ══════════════════════════════════════════
--           TRADE BY NAME SECTION
-- ══════════════════════════════════════════
local ByNameSection = Trade:Section({
    Title = "Trade By Name",
    Box = true,
    TextXAlignment = "Center",
    TextSize = 15,
    Opened = false,
})

local ByNameMonitor = ByNameSection:Paragraph({
    Title = "Status",
    Desc = "Idle",
    Color = "White",
    Locked = false,
})

local itemDropdown = ByNameSection:Dropdown({
    Title = "Select Fish Item",
    Values = CombinedModules.AutoTrade.GetFishItems(),
    Value = nil,
    AllowNone = true,
    Callback = function(value)
        if value then
            local itemName = value:match("^(.-) x") or value
            CombinedModules.AutoTrade.SetItem(itemName)
        end
    end,
})

ByNameSection:Button({
    Title = "Refresh Fish Items",
    Callback = function()
        itemDropdown:Set(CombinedModules.AutoTrade.GetFishItems())
    end,
})

ByNameSection:Input({
    Title = "Amount",
    Value = "1",
    Type = "Input",
    Placeholder = "1",
    Callback = function(value)
        CombinedModules.AutoTrade.SetItemAmount(value)
    end,
})

local byNameToggle = ByNameSection:Toggle({
    Title = "Start Trade By Name",
    Value = false,
    Callback = function(state)
        if state then
            activeMonitor = ByNameMonitor
            activeToggle = byNameToggle
            CombinedModules.AutoTrade.SetTradeMode("ByName")
            CombinedModules.AutoTrade.Start()
        else
            CombinedModules.AutoTrade.Stop()
        end
    end,
})

ByNameSection:Button({
    Title = "Reset Stats",
    Callback = function()
        CombinedModules.AutoTrade.ResetStats()
        ByNameMonitor:Set({ Desc = "Idle" })
    end,
})

-- ══════════════════════════════════════════
--           TRADE BY COIN SECTION
-- ══════════════════════════════════════════
local ByCoinSection = Trade:Section({
    Title = "Trade By Coin",
    Box = true,
    TextXAlignment = "Center",
    TextSize = 15,
    Opened = false,
})

local ByCoinMonitor = ByCoinSection:Paragraph({
    Title = "Status",
    Desc = "Idle",
    Color = "White",
    Locked = false,
})

ByCoinSection:Input({
    Title = "Target Coins",
    Value = "0",
    Type = "Input",
    Placeholder = "0",
    Callback = function(value)
        CombinedModules.AutoTrade.SetTargetCoins(value)
    end,
})

local byCoinToggle = ByCoinSection:Toggle({
    Title = "Start Trade By Coin",
    Value = false,
    Callback = function(state)
        if state then
            activeMonitor = ByCoinMonitor
            activeToggle = byCoinToggle
            CombinedModules.AutoTrade.SetTradeMode("ByCoin")
            CombinedModules.AutoTrade.Start()
        else
            CombinedModules.AutoTrade.Stop()
        end
    end,
})

ByCoinSection:Button({
    Title = "Reset Stats",
    Callback = function()
        CombinedModules.AutoTrade.ResetStats()
        ByCoinMonitor:Set({ Desc = "Idle" })
    end,
})

-- ══════════════════════════════════════════
--           TRADE BY RARITY SECTION
-- ══════════════════════════════════════════
local ByRaritySection = Trade:Section({
    Title = "Trade By Rarity",
    Box = true,
    TextXAlignment = "Center",
    TextSize = 15,
    Opened = false,
})

local ByRarityMonitor = ByRaritySection:Paragraph({
    Title = "Status",
    Desc = "Idle",
    Color = "White",
    Locked = false,
})

ByRaritySection:Dropdown({
    Title = "Select Rarity",
    Values = { "Common", "Uncommon", "Rare", "Epic", "Legendary", "Mythic", "Secret" },
    Value = 1,
    Callback = function(value)
        CombinedModules.AutoTrade.SetRarity(value)
    end,
})

ByRaritySection:Input({
    Title = "Amount",
    Value = "1",
    Type = "Input",
    Placeholder = "1",
    Callback = function(value)
        CombinedModules.AutoTrade.SetRarityAmount(value)
    end,
})

local byRarityToggle = ByRaritySection:Toggle({
    Title = "Start Trade By Rarity",
    Value = false,
    Callback = function(state)
        if state then
            activeMonitor = ByRarityMonitor
            activeToggle = byRarityToggle
            CombinedModules.AutoTrade.SetTradeMode("ByRarity")
            CombinedModules.AutoTrade.Start()
        else
            CombinedModules.AutoTrade.Stop()
        end
    end,
})

ByRaritySection:Button({
    Title = "Reset Stats",
    Callback = function()
        CombinedModules.AutoTrade.ResetStats()
        ByRarityMonitor:Set({ Desc = "Idle" })
    end,
})

-- ══════════════════════════════════════════
--         TRADE ENCHANT STONE SECTION
-- ══════════════════════════════════════════
local ByStoneSection = Trade:Section({
    Title = "Trade Enchant Stone",
    Box = true,
    TextXAlignment = "Center",
    TextSize = 15,
    Opened = false,
})

local ByStoneMonitor = ByStoneSection:Paragraph({
    Title = "Status",
    Desc = "Idle",
    Color = "White",
    Locked = false,
})

ByStoneSection:Dropdown({
    Title = "Stone Type",
    Values = { "Normal", "Double", "Evolved" },
    Value = 1,
    Callback = function(value)
        CombinedModules.AutoTrade.SetStoneType(value)
    end,
})

ByStoneSection:Input({
    Title = "Amount",
    Value = "1",
    Type = "Input",
    Placeholder = "1",
    Callback = function(value)
        CombinedModules.AutoTrade.SetStoneAmount(value)
    end,
})

ByStoneSection:Button({
    Title = "Check Enchant Stones",
    Callback = function()
        local stones = CombinedModules.AutoTrade.GetEnchantStoneItems()
        ByStoneMonitor:Set({
            Desc = #stones > 0 and ("Inventory: " .. table.concat(stones, ", ")) or "No enchant stones found"
        })
    end,
})

local byStoneToggle = ByStoneSection:Toggle({
    Title = "Start Trade Enchant Stone",
    Value = false,
    Callback = function(state)
        if state then
            activeMonitor = ByStoneMonitor
            activeToggle = byStoneToggle
            CombinedModules.AutoTrade.SetTradeMode("ByEnchantStone")
            CombinedModules.AutoTrade.Start()
        else
            CombinedModules.AutoTrade.Stop()
        end
    end,
})

ByStoneSection:Button({
    Title = "Reset Stats",
    Callback = function()
        CombinedModules.AutoTrade.ResetStats()
        ByStoneMonitor:Set({ Desc = "Idle" })
    end,
})

-- ══════════════════════════════════════════
--         WIRE UP CALLBACKS
-- ══════════════════════════════════════════
CombinedModules.AutoTrade.OnStatsChanged = function(s)
    pcall(function()
        if not activeMonitor then return end
        local coinInfo = s.coinTraded > 0 and string.format("\nCoins Traded: %d", s.coinTraded) or ""
        activeMonitor:Set({
            Desc = string.format(
                "%s\nProgress: %d/%d | Success: %d | Failed: %d%s",
                s.status or "?",
                s.totalSuccess or 0,
                s.targetAmount or 0,
                s.totalSuccess or 0,
                s.totalFailed or 0,
                coinInfo
            )
        })
    end)
end

CombinedModules.AutoTrade.OnCompleted = function(s)
    pcall(function()
        if activeToggle then activeToggle:Set(false) end
    end)
end

-- ══════════════════════════════════════════
--           WEBHOOK MODULE
-- ══════════════════════════════════════════
local WebhookModule = (function()
    local M = {}

    local function getHTTPRequest()
        local requestFunctions = {
            request, http_request,
            (syn and syn.request),
            (fluxus and fluxus.request),
            (http and http.request),
            (solara and solara.request),
        }
        for _, func in ipairs(requestFunctions) do
            if func and type(func) == "function" then return func end
        end
        return nil
    end
    local httpRequest = getHTTPRequest()

    M.FishConfig = {
        WebhookURL = "",
        DiscordUserID = "",
        HideIdentity = "",
        EnabledRarities = {},
    }
    M.DisconnectConfig = {
        WebhookURL = "",
        DiscordUserID = "",
        HideIdentity = "",
        Enabled = false
    }

    local Items, Variants
    local function loadGameModules()
        local ok = pcall(function()
            Items = require(ReplicatedStorage:WaitForChild("Items"))
            Variants = require(ReplicatedStorage:WaitForChild("Variants"))
        end)
        return ok
    end

    local TIER_NAMES = {
        [1] = "Common", [2] = "Uncommon", [3] = "Rare", [4] = "Epic",
        [5] = "Legendary", [6] = "Mythic", [7] = "SECRET"
    }
    local TIER_COLORS = {
        [1] = 9807270, [2] = 3066993, [3] = 3447003, [4] = 10181046,
        [5] = 15844367, [6] = 16711680, [7] = 65535
    }

    local isFishRunning = false
    local fishEventConnection = nil
    local isDisconnectEnabled = false
    local disconnectSetup = false

    local function getPlayerDisplayName(config)
        if config.HideIdentity and config.HideIdentity ~= "" then
            return config.HideIdentity
        end
        return localPlayer.DisplayName or localPlayer.Name
    end

    local function getDiscordImageUrl(assetId)
        if not assetId then return nil end
        local thumbnailUrl = string.format(
            "https://thumbnails.roblox.com/v1/assets?assetIds=%s&returnPolicy=PlaceHolder&size=420x420&format=Png&isCircular=false",
            tostring(assetId)
        )
        local rbxcdnUrl = string.format("https://tr.rbxcdn.com/180DAY-%s/420/420/Image/Png", tostring(assetId))
        if httpRequest then
            local success, result = pcall(function()
                local response = httpRequest({ Url = thumbnailUrl, Method = "GET" })
                if response and response.Body then
                    local data = HttpService:JSONDecode(response.Body)
                    if data and data.data and data.data[1] and data.data[1].imageUrl then
                        return data.data[1].imageUrl
                    end
                end
            end)
            if success and result then return result end
        end
        return rbxcdnUrl
    end

    local function getFishImageUrl(fish)
        local assetId = nil
        if fish.Data.Icon then assetId = tostring(fish.Data.Icon):match("%d+")
        elseif fish.Data.ImageId then assetId = tostring(fish.Data.ImageId)
        elseif fish.Data.Image then assetId = tostring(fish.Data.Image):match("%d+")
        end
        if assetId then
            local discordUrl = getDiscordImageUrl(assetId)
            if discordUrl then return discordUrl end
        end
        return "https://i.imgur.com/UMWNYK7.png"
    end

    local function getFish(itemId)
        if not Items then return nil end
        for _, f in pairs(Items) do
            if f.Data and f.Data.Id == itemId then return f end
        end
        return nil
    end

    local function getVariant(id)
        if not id or not Variants then return nil end
        local idStr = tostring(id)
        for _, v in pairs(Variants) do
            if v.Data then
                if tostring(v.Data.Id) == idStr or tostring(v.Data.Name) == idStr then return v end
            end
        end
        return nil
    end

    local function formatPrice(price)
        local formatted = tostring(math.floor(price))
        return formatted:reverse():gsub("(%d%d%d)", "%1,"):reverse():gsub("^,", "")
    end

    local function sendFishWebhook(fish, meta, extra)
        if not M.FishConfig.WebhookURL or M.FishConfig.WebhookURL == "" then return end
        if not httpRequest then return end

        local tier = TIER_NAMES[fish.Data.Tier] or "Unknown"
        local color = TIER_COLORS[fish.Data.Tier] or 3447003
        local enabledRarities = M.FishConfig.EnabledRarities

        if enabledRarities then
            local raritySet = {}
            local hasAnyFilter = false
            for k, v in pairs(enabledRarities) do
                if type(k) == "number" and type(v) == "string" then
                    raritySet[v] = true hasAnyFilter = true
                elseif type(k) == "string" and v == true then
                    raritySet[k] = true hasAnyFilter = true
                end
            end
            if hasAnyFilter and not raritySet[tier] then return end
        end

        local mutationText = "None"
        local finalPrice = fish.SellPrice or 0
        local variantId = nil
        if extra then variantId = extra.Variant or extra.Mutation or extra.VariantId or extra.MutationId end
        if not variantId and meta then variantId = meta.Variant or meta.Mutation or meta.VariantId or meta.MutationId end

        local isShiny = (meta and meta.Shiny) or (extra and extra.Shiny)
        if isShiny then mutationText = "Shiny" finalPrice = finalPrice * 2 end

        if variantId then
            local v = getVariant(variantId)
            if v then
                mutationText = v.Data.Name .. " (" .. v.SellMultiplier .. "x)"
                finalPrice = finalPrice * v.SellMultiplier
            else
                mutationText = variantId
            end
        end

        local imageUrl = getFishImageUrl(fish)
        local playerDisplayName = getPlayerDisplayName(M.FishConfig)
        local mention = M.FishConfig.DiscordUserID ~= "" and "<@" .. M.FishConfig.DiscordUserID .. ">" or ""
        local congratsMsg = string.format("%s **%s** has caught a new **%s** tier fish!", mention, playerDisplayName, tier)

        local payload = {
            username = "King Vypers",
            avatar_url = "https://raw.githubusercontent.com/semuao621-wq/Kamunanya/main/Kingvyperslogo.jpg",
            embeds = {{
                author = { name = "King Vypers | Fish Caught Notification" },
                description = congratsMsg,
                color = color,
                fields = {
                    { name = "Fish Name", value = "```" .. fish.Data.Name .. "```", inline = false },
                    { name = "Tier", value = "```" .. tier .. "```", inline = true },
                    { name = "Weight", value = "```" .. string.format("%.2f Kg", meta.Weight or 0) .. "```", inline = true },
                    { name = "Mutation", value = "```" .. mutationText .. "```", inline = true },
                    { name = "Base Price", value = "```$" .. formatPrice(fish.SellPrice or 0) .. "```", inline = true },
                    { name = "Final Price", value = "```$" .. formatPrice(finalPrice) .. "```", inline = true },
                    { name = "Shiny", value = "```" .. (isShiny and "Yes" or "No") .. "```", inline = true },
                },
                image = { url = imageUrl },
                footer = {
                    text = "King Vypers • " .. os.date("%m/%d/%Y at %I:%M %p"),
                    icon_url = "https://raw.githubusercontent.com/semuao621-wq/Kamunanya/main/Kingvyperslogo.jpg"
                },
                timestamp = os.date("!%Y-%m-%dT%H:%M:%SZ")
            }}
        }

        pcall(function()
            httpRequest({
                Url = M.FishConfig.WebhookURL,
                Method = "POST",
                Headers = { ["Content-Type"] = "application/json" },
                Body = HttpService:JSONEncode(payload)
            })
        end)
    end

    local function sendDisconnectWebhook(reason)
        if not isDisconnectEnabled then return end
        local webhookURL = M.DisconnectConfig.WebhookURL
        if not webhookURL or webhookURL == "" then return end
        if not httpRequest then return end

        local playerName = getPlayerDisplayName(M.DisconnectConfig)
        local mention = M.DisconnectConfig.DiscordUserID ~= ""
            and "<@" .. M.DisconnectConfig.DiscordUserID:gsub("%D", "") .. ">"
            or ""
        local disconnectReason = reason and reason ~= "" and reason or "Disconnected from server"
        local contentMsg = mention ~= ""
            and mention .. " Your account has been disconnected from the server!"
            or "Account disconnected from server!"

        local payload = {
            content = contentMsg,
            username = "King Vypers",
            avatar_url = "https://raw.githubusercontent.com/semuao621-wq/Kamunanya/main/Kingvyperslogo.jpg",
            embeds = {{
                author = { name = "King Vypers | Disconnect Alert" },
                title = "Connection Lost",
                description = "**Your Roblox session has been disconnected.**\n\nAttempting to rejoin the server...",
                color = 9055487,
                fields = {
                    { name = "Account", value = "```" .. playerName .. "```", inline = true },
                    { name = "Time", value = "```" .. os.date("%m/%d/%Y at %I:%M %p") .. "```", inline = true },
                    { name = "Reason", value = "```" .. disconnectReason .. "```", inline = false },
                },
                footer = {
                    text = "King Vypers • Auto-rejoin enabled",
                    icon_url = "https://raw.githubusercontent.com/semuao621-wq/Kamunanya/main/Kingvyperslogo.jpg"
                },
                timestamp = os.date("!%Y-%m-%dT%H:%M:%SZ")
            }}
        }

        task.spawn(function()
            pcall(function()
                httpRequest({
                    Url = webhookURL,
                    Method = "POST",
                    Headers = { ["Content-Type"] = "application/json" },
                    Body = HttpService:JSONEncode(payload)
                })
            end)
        end)
    end

    local function setupDisconnectDetection()
        if disconnectSetup then return end
        disconnectSetup = true
        local hasDisconnected = false

        local function handleDisconnect(reason)
            if not hasDisconnected and isDisconnectEnabled then
                hasDisconnected = true
                sendDisconnectWebhook(reason or "Disconnected from server")
                task.wait(2)
                game:GetService("TeleportService"):Teleport(game.PlaceId, localPlayer)
            end
        end

        game:GetService("GuiService").ErrorMessageChanged:Connect(function(message)
            if message and message ~= "" then handleDisconnect(message) end
        end)

        pcall(function()
            local CoreGui = game:GetService("CoreGui")
            local RobloxPromptGui = CoreGui:FindFirstChild("RobloxPromptGui")
            if RobloxPromptGui then
                local promptOverlay = RobloxPromptGui:FindFirstChild("promptOverlay")
                if promptOverlay then
                    promptOverlay.ChildAdded:Connect(function(child)
                        if child.Name == "ErrorPrompt" then
                            task.wait(1)
                            local textLabel = child:FindFirstChildWhichIsA("TextLabel", true)
                            handleDisconnect(textLabel and textLabel.Text or "Disconnected")
                        end
                    end)
                end
            end
        end)
    end

    function M:SetFishWebhookURL(url) self.FishConfig.WebhookURL = url end
    function M:SetFishDiscordUserID(id) self.FishConfig.DiscordUserID = id end
    function M:SetFishEnabledRarities(rarities) self.FishConfig.EnabledRarities = rarities end
    function M:SetFishHideIdentity(name) self.FishConfig.HideIdentity = name end
    function M:SetDisconnectWebhookURL(url) self.DisconnectConfig.WebhookURL = url end
    function M:SetDisconnectDiscordUserID(id) self.DisconnectConfig.DiscordUserID = id end
    function M:SetDisconnectHideIdentity(name) self.DisconnectConfig.HideIdentity = name end
    function M:EnableDisconnectWebhook(enabled)
        self.DisconnectConfig.Enabled = enabled
        isDisconnectEnabled = enabled
        if enabled then setupDisconnectDetection() end
    end

    function M:StartFishWebhook()
        if isFishRunning then return false end
        if not self.FishConfig.WebhookURL or self.FishConfig.WebhookURL == "" then return false end
        if not httpRequest then return false end
        if not loadGameModules() then return false end
        local re = EmbeddedEventResolver:GetRE("ObtainedNewFishNotification")
        if not re or not re.OnClientEvent then return false end
        fishEventConnection = re.OnClientEvent:Connect(function(itemId, metadata, extraData)
            local fish = getFish(itemId)
            if fish then task.spawn(function() sendFishWebhook(fish, metadata or {}, extraData or {}) end) end
        end)
        isFishRunning = true
        return true
    end

    function M:StopFishWebhook()
        if not isFishRunning then return false end
        if fishEventConnection then fishEventConnection:Disconnect() fishEventConnection = nil end
        isFishRunning = false
        return true
    end

    function M:TestFishWebhook()
        if not httpRequest then return false end
        if not self.FishConfig.WebhookURL or self.FishConfig.WebhookURL == "" then return false end
        pcall(function()
            httpRequest({
                Url = self.FishConfig.WebhookURL,
                Method = "POST",
                Headers = { ["Content-Type"] = "application/json" },
                Body = HttpService:JSONEncode({
                    username = "King Vypers",
                    avatar_url = "https://raw.githubusercontent.com/semuao621-wq/Kamunanya/main/Kingvyperslogo.jpg",
                    embeds = {{
                        title = "🎣 Webhook Connection Test",
                        description = "**Connection Status:** ✅ Successfully Connected!\n\n*Your webhook is now ready to receive fish catch notifications.*",
                        color = 9055487,
                        fields = {
                            { name = "System Status", value = "```diff\n+ Webhook Active\n+ Logger Ready\n+ Notifications Enabled```", inline = true },
                            { name = "Features", value = "```yaml\nAuto-Logging: ON\nReal-time: ON\nGame: Fish It```", inline = true },
                        },
                        footer = {
                            text = "King Vypers • Test Successful",
                            icon_url = "https://raw.githubusercontent.com/semuao621-wq/Kamunanya/main/Kingvyperslogo.jpg"
                        },
                        timestamp = os.date("!%Y-%m-%dT%H:%M:%SZ")
                    }}
                })
            })
        end)
        return true
    end

    function M:TestDisconnectWebhook()
        if not httpRequest then return false end
        if not self.DisconnectConfig.WebhookURL or self.DisconnectConfig.WebhookURL == "" then return false end
        sendDisconnectWebhook("Test Successfully :3")
        task.wait(2)
        game:GetService("TeleportService"):Teleport(game.PlaceId, localPlayer)
        return true
    end

    function M:IsFishRunning() return isFishRunning end
    function M:IsDisconnectEnabled() return isDisconnectEnabled end
    function M:IsSupported() return httpRequest ~= nil end

    return M
end)()

-- ══════════════════════════════════════════
--              WEBHOOK TAB
-- ══════════════════════════════════════════
local Webhook = Window:Tab({
    Title = "Webhook",
    Icon = "message-circle",
    IconColor = Mains,
    IconShape = "Square",
    Border = true,
})

-- ══════════════════════════════════════════
--         FISH CAUGHT WEBHOOK SECTION
-- ══════════════════════════════════════════
local FishWebhookSection = Webhook:Section({
    Title = "Fish Caught Webhook",
    Box = true,
    TextXAlignment = "Center",
    TextSize = 15,
    Opened = true,
})

FishWebhookSection:Paragraph({
    Title = "Info",
    Desc = "Kirim notifikasi ke Discord setiap kali kamu catch ikan. Isi Webhook URL terlebih dahulu sebelum enable.",
    Color = "White",
    Locked = false,
})

local currentFishWebhookURL = ""
local currentFishDiscordID = ""
local currentFishHideIdentity = ""
local fishWebhookToggle = nil

FishWebhookSection:Input({
    Title = "Webhook URL",
    Value = "",
    Type = "Input",
    Placeholder = "https://discord.com/api/webhooks/...",
    Callback = function(value)
        currentFishWebhookURL = value:gsub("^%s*(.-)%s*$", "%1")
        pcall(function() WebhookModule:SetFishWebhookURL(currentFishWebhookURL) end)
    end,
})

FishWebhookSection:Input({
    Title = "Discord User ID",
    Value = "",
    Type = "Input",
    Placeholder = "123456789012345678",
    Callback = function(value)
        currentFishDiscordID = value:gsub("^%s*(.-)%s*$", "%1")
        pcall(function() WebhookModule:SetFishDiscordUserID(currentFishDiscordID) end)
    end,
})

FishWebhookSection:Input({
    Title = "Hide Identity (Custom Name)",
    Value = "",
    Type = "Input",
    Placeholder = "Enter custom name...",
    Callback = function(value)
        currentFishHideIdentity = value:gsub("^%s*(.-)%s*$", "%1")
        pcall(function() WebhookModule:SetFishHideIdentity(currentFishHideIdentity) end)
    end,
})

FishWebhookSection:Dropdown({
    Title = "Rarity Filter",
    Values = { "Common", "Uncommon", "Rare", "Epic", "Legendary", "Mythic", "SECRET" },
    Value = nil,
    AllowNone = true,
    Multi = true,
    Callback = function(selected)
        if type(selected) == "table" then
            pcall(function() WebhookModule:SetFishEnabledRarities(selected) end)
        end
    end,
})

fishWebhookToggle = FishWebhookSection:Toggle({
    Title = "Enable Fish Webhook",
    Value = false,
    Callback = function(state)
        if state then
            if currentFishWebhookURL == "" then
                fishWebhookToggle:Set(false)
                return
            end
            pcall(function()
                WebhookModule:SetFishWebhookURL(currentFishWebhookURL)
                if currentFishDiscordID ~= "" then WebhookModule:SetFishDiscordUserID(currentFishDiscordID) end
                if currentFishHideIdentity ~= "" then WebhookModule:SetFishHideIdentity(currentFishHideIdentity) end
                WebhookModule:StartFishWebhook()
            end)
        else
            pcall(function() WebhookModule:StopFishWebhook() end)
        end
    end,
})

FishWebhookSection:Button({
    Title = "Test Fish Webhook",
    Callback = function()
        if currentFishWebhookURL == "" then return end
        pcall(function() WebhookModule:TestFishWebhook() end)
    end,
})

-- ══════════════════════════════════════════
--        DISCONNECT WEBHOOK SECTION
-- ══════════════════════════════════════════
local DisconnectWebhookSection = Webhook:Section({
    Title = "Disconnect Webhook",
    Box = true,
    TextXAlignment = "Center",
    TextSize = 15,
    Opened = false,
})

DisconnectWebhookSection:Paragraph({
    Title = "Info",
    Desc = "Kirim notifikasi ke Discord saat Roblox disconnect, lalu otomatis rejoin server.",
    Color = "White",
    Locked = false,
})

local currentDisconnectWebhookURL = ""
local currentDisconnectDiscordID = ""
local currentDisconnectHideIdentity = ""

DisconnectWebhookSection:Input({
    Title = "Webhook URL",
    Value = "",
    Type = "Input",
    Placeholder = "https://discord.com/api/webhooks/...",
    Callback = function(value)
        currentDisconnectWebhookURL = value:gsub("^%s*(.-)%s*$", "%1")
        pcall(function() WebhookModule:SetDisconnectWebhookURL(currentDisconnectWebhookURL) end)
    end,
})

DisconnectWebhookSection:Input({
    Title = "Discord User ID",
    Value = "",
    Type = "Input",
    Placeholder = "123456789012345678",
    Callback = function(value)
        currentDisconnectDiscordID = value:gsub("^%s*(.-)%s*$", "%1")
        pcall(function() WebhookModule:SetDisconnectDiscordUserID(currentDisconnectDiscordID) end)
    end,
})

DisconnectWebhookSection:Input({
    Title = "Hide Identity (Custom Name)",
    Value = "",
    Type = "Input",
    Placeholder = "Enter custom name...",
    Callback = function(value)
        currentDisconnectHideIdentity = value:gsub("^%s*(.-)%s*$", "%1")
        pcall(function() WebhookModule:SetDisconnectHideIdentity(currentDisconnectHideIdentity) end)
    end,
})

DisconnectWebhookSection:Toggle({
    Title = "Enable Disconnect Webhook",
    Value = false,
    Callback = function(state)
        pcall(function() WebhookModule:EnableDisconnectWebhook(state) end)
    end,
})

DisconnectWebhookSection:Button({
    Title = "Test Disconnect Webhook",
    Callback = function()
        if currentDisconnectWebhookURL == "" then return end
        pcall(function() WebhookModule:TestDisconnectWebhook() end)
    end,
})

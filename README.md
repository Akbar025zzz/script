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
})

-- */  Colors  /* --
local Kings = Color3.fromHex("#120324")
local Mains = Color3.fromHex("#110029")
local Purple = Color3.fromHex("#7775F2")
local Yellow = Color3.fromHex("#ECA201")
local Green = Color3.fromHex("#10C550")
local Grey = Color3.fromHex("#292828")
local Blue = Color3.fromHex("#257AF7")
local Red = Color3.fromHex("#EF4F1D")


-- TARUH DISINI â†“
WindUI:AddTheme({
    Name = "MachTheme",
    Background = Kings,
})
WindUI:SetTheme("MachTheme")
-- SAMPAI SINI â†‘

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
-- ðŸ”´ TOMBOL MERAH (PC + MOBILE SUPPORT)
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
-- ðŸ–±ï¸ðŸ“± DRAG + CLICK (PC + MOBILE)
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
                    print("ðŸ”„ Window toggled")
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
    Desc = "â€¢ Member Count: " .. memberCount .. "\nâ€¢ Online Count: " .. onlineCount,
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
                ServerInfo:SetDesc("â€¢ Member Count: " .. memberCount .. "\nâ€¢ Online Count: " .. onlineCount)
            end
        }
    }
})

-- FISHING TAB
local FarmTab = Window:Tab({
    Title = "Farm",
    Icon = "fish",
	IconColor = Mains,
	IconShape = "Square",
	Border = true,
})

-- == INSTANT FISHING LOGIC ==
local RS = game:GetService("ReplicatedStorage")
local LP = Players.LocalPlayer

-- Initialize Knit services safely
local Knit, FishingRF, RewardRF, RewardRE
local START_FISHING, THROW_FLOATER, CONFIRM_CAST, REQUEST_FISH_BITE, START_PULLING, FISHING_PULL_INPUT, STOP_FISHING, PULL_STATE_EVENT

task.spawn(function()
    pcall(function()
        Knit = RS.Packages._Index["sleitnick_knit@1.7.0"].knit.Services
        FishingRF = Knit.FishingReplicationService.RF
        RewardRF  = Knit.FishingRewardService.RF
        RewardRE  = Knit.FishingRewardService.RE

        START_FISHING      = FishingRF.StartFishing
        THROW_FLOATER      = FishingRF.ThrowFloater
        CONFIRM_CAST       = FishingRF.ConfirmFloatingCast
        REQUEST_FISH_BITE  = RewardRF.RequestFishBite
        START_PULLING      = FishingRF.StartPulling
        FISHING_PULL_INPUT = RewardRF.FishingPullInput
        STOP_FISHING       = FishingRF.StopFishing
        PULL_STATE_EVENT   = RewardRE:WaitForChild("FishingPullState")
    end)
end)

local FLOATER = "Floater_Doll"
local FLOATER_PROPS = {
    LightInfluence = 1,
    Transparency   = 0.15,
    Color          = Color3.new(1, 0.882, 0.207),
    FaceCamera     = true,
    LightEmission  = 1,
    Width          = 0.12
}

local autoInstantFish = false
local castDistValue = 28
local throwPowerValue = 10
local fishDelayValue = 3

local function getCastPosAndRod(castDist)
    local char = LP.Character or LP.CharacterAdded:Wait()
    local hrp = char:WaitForChild("HumanoidRootPart")
    local playerPos = hrp.Position
    local lookDir = hrp.CFrame.LookVector
    
    local targetXZ = playerPos + (lookDir * castDist)
    local rayParams = RaycastParams.new()
    rayParams.FilterDescendantsInstances = {char}
    rayParams.FilterType = Enum.RaycastFilterType.Exclude
    rayParams.IgnoreWater = false
    
    local origin = targetXZ + Vector3.new(0, 15, 0)
    local raycastResult = workspace:Raycast(origin, Vector3.new(0, -100, 0), rayParams)
    
    local castPos
    if raycastResult then
        castPos = raycastResult.Position
    else
        castPos = targetXZ + Vector3.new(0, -8, 0)
    end
    
    local tool = char:FindFirstChildOfClass("Tool")
    local currentRod = tool and tool.Name or "BananaRod"
    
    return playerPos, castPos, currentRod
end

local function doInstantFishCycle()
    if not START_FISHING then return end
    
    local playerPos, castPos, currentRod = getCastPosAndRod(castDistValue)
    print("ðŸŽ£ INSTANT FISHING - SIKLUS DIMULAI!")
    print("ðŸ“ Cast ke:", castPos, "| Rod:", currentRod)
    
    local fishingDone = false
    local activeSessionId = nil
    
    local conn
    if PULL_STATE_EVENT then
        conn = PULL_STATE_EVENT.OnClientEvent:Connect(function(data)
            if type(data) == "table" and data.sessionId then
                print("ðŸ“¥ EVENT DARI SERVER -> Type:", tostring(data.type), "| Reason:", tostring(data.reason))
                if data.type == "resolved" then
                    if data.sessionId == activeSessionId then
                        fishingDone = true
                        print("ðŸ RESOLVED! Ikan masuk/gagal:", data.reason, "| Success:", data.success)
                    end
                end
            end
        end)
    end

    print("0ï¸âƒ£-3ï¸âƒ£ Fire semua sekaligus (parallel)...")

    task.spawn(function() pcall(function() STOP_FISHING:InvokeServer() end) end)
    task.spawn(function() pcall(function() START_FISHING:InvokeServer(currentRod, FLOATER) end) end)
    task.spawn(function() pcall(function() THROW_FLOATER:InvokeServer(playerPos, castPos, currentRod, FLOATER, FLOATER_PROPS, throwPowerValue) end) end)
    task.spawn(function() pcall(function() CONFIRM_CAST:InvokeServer(castPos) end) end)

    if not autoInstantFish then if conn then conn:Disconnect() end return end
    print("4ï¸âƒ£ RequestFishBite...")
    local sessionId = nil
    local s4, r4 = pcall(function() return REQUEST_FISH_BITE:InvokeServer(castPos) end)
    if s4 and type(r4) == "table" then
        sessionId = r4.SessionId
        activeSessionId = sessionId
        print("ðŸŒŸ Session ID:", sessionId)
    else
        print("âŒ RequestFishBite gagal:", tostring(r4))
    end

    if not autoInstantFish then if conn then conn:Disconnect() end return end

    print("â³ Nunggu " .. tostring(fishDelayValue) .. " detik biar ikan beneran gigit & server ready...")
    task.wait(fishDelayValue)
    
    if not autoInstantFish then if conn then conn:Disconnect() end return end

    print("5ï¸âƒ£ StartPulling...")
    pcall(function() START_PULLING:InvokeServer() end)

    if sessionId then
        print("6ï¸âƒ£ SPAM TAP! UUID:", sessionId)
        print("âž¡ï¸ Kirim 'begin'...")
        pcall(function() FISHING_PULL_INPUT:InvokeServer(sessionId, "begin") end)

        local waitStart = tick()
        local tapCount = 0
        while not fishingDone and tick() - waitStart < 15 and autoInstantFish do
            for i = 1, 2 do
                task.spawn(function()
                    pcall(function() FISHING_PULL_INPUT:InvokeServer(sessionId, "tap") end)
                end)
            end
            tapCount = tapCount + 2
            RunService.Heartbeat:Wait()
        end
        print("ðŸŸ Spam berhenti! Total tap:", tapCount)
    else
        print("âŒ Session ID nil, cast gagal")
    end

    if conn then conn:Disconnect() end

    -- STOP FISHING DI AKHIR
    print("ðŸ›‘ StopFishing (finish)...")
    pcall(function() STOP_FISHING:InvokeServer() end)
    print("ðŸ Script Selesai!")
end

-- == UI ELEMENTS ==
FarmTab:Toggle({
    Title = "Auto Instant Fish",
    Desc = "Mancing + tap instant otomatis",
    Value = false,
    Callback = function(state)
        autoInstantFish = state
        if state then
            task.spawn(function()
                while autoInstantFish do
                    doInstantFishCycle()
                end
            end)
        end
    end,
})

FarmTab:Input({
    Title = "Cast Distance",
    Desc = "Jarak lemparan pancingan",
    Value = "28",
    Placeholder = "28",
    Callback = function(text)
        local num = tonumber(text)
        if num then
            castDistValue = num
        end
    end,
})

FarmTab:Input({
    Title = "Throw Power",
    Desc = "Kekuatan melempar",
    Value = "10",
    Placeholder = "10",
    Callback = function(text)
        local num = tonumber(text)
        if num then
            throwPowerValue = num
        end
    end,
})

FarmTab:Input({
    Title = "Fish Delay",
    Desc = "Delay nunggu server (detik)",
    Value = "3",
    Placeholder = "3",
    Callback = function(text)
        local num = tonumber(text)
        if num then
            fishDelayValue = num
        end
    end,
})

InfoTab:Select()

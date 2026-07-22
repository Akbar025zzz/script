-- ============================================================================
-- // KING V YP E R S - AUTO INSTANT FISHING (LEMPAR - TARIK - DAPAT)
-- ============================================================================
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
    OpenButton = { Enabled = false },
})

-- Colors
local Kings  = Color3.fromHex("#120324")
local Mains  = Color3.fromHex("#110029")
local Purple = Color3.fromHex("#7775F2")

WindUI:AddTheme({
    Name = "MachTheme",
    Background = Kings,
})
WindUI:SetTheme("MachTheme")

Window:Tag({ Title = "PREMIUM", Color = Mains })
Window:Tag({ Title = "BETA", Color = Purple })

-- =================================================================
-- TOMBOL TOGGLE GUI (DRAG & CLICK SUPPORT PC + MOBILE)
-- =================================================================
local Players = game:GetService("Players")
local TweenService = game:GetService("TweenService")
local UserInputService = game:GetService("UserInputService")
local CoreGui = game:GetService("CoreGui")

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

protectGui = success and result or Players.LocalPlayer:WaitForChild("PlayerGui")

local screenGui = Instance.new("ScreenGui")
screenGui.Name = "MachFishingButton"
screenGui.ResetOnSpawn = false
screenGui.Parent = protectGui

local buttonFrame = Instance.new("Frame")
buttonFrame.Size = UDim2.new(0, 42, 0, 42)
buttonFrame.Position = UDim2.new(0, 20, 0, 20)
buttonFrame.BackgroundTransparency = 1
buttonFrame.Parent = screenGui

local imageButton = Instance.new("ImageButton")
imageButton.Size = UDim2.new(1, 0, 1, 0)
imageButton.BackgroundColor3 = Color3.fromRGB(20, 20, 20)
imageButton.BackgroundTransparency = 0.2
imageButton.Image = "rbxassetid://107726435417936"
imageButton.Parent = buttonFrame

local uiCorner = Instance.new("UICorner"); uiCorner.CornerRadius = UDim.new(0, 8); uiCorner.Parent = imageButton
local uiStroke = Instance.new("UIStroke"); uiStroke.Thickness = 2; uiStroke.Color = Color3.fromRGB(60, 60, 60); uiStroke.Parent = imageButton

local dragging, dragStart, startPos, dragInput
imageButton.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
        dragging = true
        dragStart = input.Position
        startPos = buttonFrame.Position
    end
end)

imageButton.InputChanged:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch then
        dragInput = input
    end
end)

UserInputService.InputChanged:Connect(function(input)
    if dragging and dragInput and (input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch) then
        local delta = input.Position - dragStart
        buttonFrame.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
    end
end)

local clickStart = nil
imageButton.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
        clickStart = input.Position
    end
end)

imageButton.InputEnded:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
        if clickStart then
            if (input.Position - clickStart).Magnitude < 10 then
                if Window and Window.Toggle then Window:Toggle() end
            end
            clickStart = nil
            dragging = false
        end
    end
end)

-- =================================================================
-- INFO TAB
-- =================================================================
local InfoTab = Window:Tab({ Title = "Info", Icon = "solar:info-square-bold", Border = true })
local memberCount, onlineCount = "N/A", "N/A"

local function fetchDiscordInfo()
    local req = request or http_request or (syn and syn.request)
    if not req then return end
    pcall(function()
        local res = req({ Url = "https://discord.com/api/v9/invites/XmWf3YQPpZ?with_counts=true", Method = "GET" })
        if res and res.StatusCode == 200 then
            local data = game:GetService("HttpService"):JSONDecode(res.Body)
            memberCount = tostring(data.approximate_member_count or "N/A")
            onlineCount = tostring(data.approximate_presence_count or "N/A")
        end
    end)
end

fetchDiscordInfo()

local ServerInfo = InfoTab:Paragraph({
    Title = "King Vypers | Official",
    Desc = "• Member Count: " .. memberCount .. "\n• Online Count: " .. onlineCount,
    Image = "rbxassetid://107726435417936",
    Thumbnail = "rbxassetid://83197533072664",
    ThumbnailSize = 80,
    Buttons = {
        {
            Title = "Copy Discord Invite",
            Color = Color3.fromHex("#5707AB"),
            Icon = "link",
            Callback = function()
                if setclipboard then setclipboard("https://discord.gg/XmWf3YQPpZ") end
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

-- =================================================================
-- FARM TAB (EXCESSIVE INSTANT FISHING)
-- =================================================================
local FarmTab = Window:Tab({ Title = "Farm", Icon = "fish", Border = true })

local RS = game:GetService("ReplicatedStorage")
local LP = Players.LocalPlayer

local Knit, FishingRF, RewardRF
local START_FISHING, THROW_FLOATER, CONFIRM_CAST, REQUEST_FISH_BITE, START_PULLING, FISHING_PULL_INPUT, STOP_FISHING

task.spawn(function()
    pcall(function()
        Knit = RS.Packages._Index["sleitnick_knit@1.7.0"].knit.Services
        FishingRF = Knit.FishingReplicationService.RF
        RewardRF  = Knit.FishingRewardService.RF

        START_FISHING      = FishingRF.StartFishing
        THROW_FLOATER      = FishingRF.ThrowFloater
        CONFIRM_CAST       = FishingRF.ConfirmFloatingCast
        REQUEST_FISH_BITE  = RewardRF.RequestFishBite
        START_PULLING      = FishingRF.StartPulling
        FISHING_PULL_INPUT = RewardRF.FishingPullInput
        STOP_FISHING       = FishingRF.StopFishing
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
local fishDelayValue = 0.01

local function getCastPosAndRod(castDist)
    local char = LP.Character or LP.CharacterAdded:Wait()
    local hrp = char:WaitForChild("HumanoidRootPart")
    local playerPos = hrp.Position
    local lookDir = hrp.CFrame.LookVector
    
    local targetXZ = playerPos + (lookDir * castDist)
    local origin = targetXZ + Vector3.new(0, 15, 0)
    
    local rayParams = RaycastParams.new()
    rayParams.FilterDescendantsInstances = {char}
    rayParams.FilterType = Enum.RaycastFilterType.Exclude
    rayParams.IgnoreWater = false
    
    local raycastResult = workspace:Raycast(origin, Vector3.new(0, -100, 0), rayParams)
    local castPos = raycastResult and raycastResult.Position or (targetXZ + Vector3.new(0, -8, 0))
    
    local tool = char:FindFirstChildOfClass("Tool")
    local currentRod = tool and tool.Name or "BananaRod"
    
    return playerPos, castPos, currentRod
end

-- FUNGSIONALITAS UTAMA: LEMPAR - TARIK - DAPAT
local function doInstantCycle()
    if not START_FISHING then return end

    local playerPos, castPos, currentRod = getCastPosAndRod(castDistValue)

    -- 1. LEMPAR
    pcall(function() STOP_FISHING:InvokeServer() end)
    pcall(function() START_FISHING:InvokeServer(currentRod, FLOATER) end)
    pcall(function() THROW_FLOATER:InvokeServer(playerPos, castPos, currentRod, FLOATER, FLOATER_PROPS, throwPowerValue) end)
    pcall(function() CONFIRM_CAST:InvokeServer(castPos) end)

    -- 2. TARIK
    local ok, res = pcall(function() return REQUEST_FISH_BITE:InvokeServer(castPos) end)
    local sessionId = (ok and type(res) == "table") and res.SessionId or nil

    pcall(function() START_PULLING:InvokeServer() end)

    -- 3. DAPAT (Burst Input Tap)
    if sessionId then
        pcall(function() FISHING_PULL_INPUT:InvokeServer(sessionId, "begin") end)
        
        for _ = 1, 10 do
            task.spawn(function()
                pcall(function() FISHING_PULL_INPUT:InvokeServer(sessionId, "tap") end)
            end)
        end
    end

    pcall(function() STOP_FISHING:InvokeServer() end)
end

-- =================================================================
-- UI CONTROL ELEMENTS
-- =================================================================
FarmTab:Toggle({
    Title = "Auto Instant Fish (Fast)",
    Desc = "Mode Ekstrim: Lempar -> Tarik -> Dapat",
    Value = false,
    Callback = function(state)
        autoInstantFish = state
        if state then
            task.spawn(function()
                while autoInstantFish do
                    doInstantCycle()
                    task.wait(fishDelayValue)
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
        if num then castDistValue = num end
    end,
})

FarmTab:Input({
    Title = "Throw Power",
    Desc = "Kekuatan melempar",
    Value = "10",
    Placeholder = "10",
    Callback = function(text)
        local num = tonumber(text)
        if num then throwPowerValue = num end
    end,
})

FarmTab:Input({
    Title = "Fish Delay",
    Desc = "Micro delay siklus (Default 0.01s)",
    Value = "0.01",
    Placeholder = "0.01",
    Callback = function(text)
        local num = tonumber(text)
        if num then fishDelayValue = num end
    end,
})

InfoTab:Select()

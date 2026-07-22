-- ============================================================================
-- // KING V YP E R S - FORCE BITE + F9 DEBUG LOGGING (FISH IT)
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

WindUI:AddTheme({ Name = "MachTheme", Background = Kings })
WindUI:SetTheme("MachTheme")

Window:Tag({ Title = "PREMIUM", Color = Mains })
Window:Tag({ Title = "FORCE BITE + LOG", Color = Purple })

-- =================================================================
-- TOMBOL TOGGLE GUI (DRAG & CLICK)
-- =================================================================
local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local CoreGui = game:GetService("CoreGui")

local protectGui
local success, result = pcall(function()
    if gethui then return gethui()
    elseif syn and syn.protect_gui then
        local sg = Instance.new("ScreenGui")
        syn.protect_gui(sg)
        sg.Parent = CoreGui
        return sg.Parent
    else return CoreGui end
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
        dragging = true; dragStart = input.Position; startPos = buttonFrame.Position
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
        if clickStart and (input.Position - clickStart).Magnitude < 10 then
            if Window and Window.Toggle then Window:Toggle() end
        end
        clickStart = nil; dragging = false
    end
end)

-- =================================================================
-- INFO TAB
-- =================================================================
local InfoTab = Window:Tab({ Title = "Info", Icon = "solar:info-square-bold", Border = true })
InfoTab:Paragraph({
    Title = "King Vypers | Official",
    Desc = "Script Optimized for Fish It.\n• Mode: Force Instant Bite\n• Cek F9 Console untuk melihat Log Tangkapan Ikan.",
    Image = "rbxassetid://107726435417936",
    Thumbnail = "rbxassetid://83197533072664",
    ThumbnailSize = 80,
})

-- =================================================================
-- FARM TAB & BYPASS CORE
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
    LightInfluence = 1, Transparency = 0.15, Color = Color3.new(1, 0.882, 0.207),
    FaceCamera = true, LightEmission = 1, Width = 0.12
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

-- =================================================================
-- INTI BYPASS (DENGAN LOGGING F9)
-- =================================================================
local cycleCount = 0

local function doForceBiteCycle()
    if not START_FISHING then return end
    cycleCount = cycleCount + 1

    local playerPos, castPos, currentRod = getCastPosAndRod(castDistValue)

    task.spawn(function()
        print("[King Vypers] 🎣 --- Memulai Siklus ke-" .. tostring(cycleCount) .. " ---")
        
        -- 1. Lempar Umpan
        pcall(function() STOP_FISHING:InvokeServer() end)
        pcall(function() START_FISHING:InvokeServer(currentRod, FLOATER) end)
        pcall(function() THROW_FLOATER:InvokeServer(playerPos, castPos, currentRod, FLOATER, FLOATER_PROPS, throwPowerValue) end)
        pcall(function() CONFIRM_CAST:InvokeServer(castPos) end)

        -- 2. PAKSA MAKAN
        local ok, res = pcall(function() return REQUEST_FISH_BITE:InvokeServer(castPos) end)
        
        if ok and type(res) == "table" then
            local sessionId = res.SessionId
            if sessionId then
                warn("[King Vypers] ✅ UMPAN DIMAKAN! SessionID: " .. tostring(sessionId))
                
                -- Nge-print semua data yang dikasih dari server (jenis ikan, rarity, dll kalau ada)
                for k, v in pairs(res) do
                    print("   -> " .. tostring(k) .. ": " .. tostring(v))
                end

                -- 3. LANGSUNG TARIK
                pcall(function() START_PULLING:InvokeServer() end)
                pcall(function() FISHING_PULL_INPUT:InvokeServer(sessionId, "begin") end)
                
                print("[King Vypers] ⚡ Mengeksekusi 15x Tap Instan untuk Bypass Tarikan...")
                for _ = 1, 15 do
                    task.spawn(function()
                        pcall(function() FISHING_PULL_INPUT:InvokeServer(sessionId, "tap") end)
                    end)
                end
                warn("[King Vypers] 🐟 IKAN BERHASIL DITANGKAP!")
            else
                warn("[King Vypers] ❌ MISS! Ikan tidak merespon umpan atau data kosong.")
            end
        else
            warn("[King Vypers] ⚠️ GAGAL REQUEST! Server tidak merespon gigitan ikan.")
        end

        print("[King Vypers] 🛑 Reset Pancingan...")
        print("------------------------------------------------")
        pcall(function() STOP_FISHING:InvokeServer() end)
    end)
end

-- =================================================================
-- UI CONTROL ELEMENTS
-- =================================================================
FarmTab:Toggle({
    Title = "Auto Farm (Force Bite + Log)",
    Desc = "Buka F9 Console untuk melihat Log pergerakan ikan",
    Value = false,
    Callback = function(state)
        autoInstantFish = state
        if state then
            task.spawn(function()
                while autoInstantFish do
                    doForceBiteCycle()
                    task.wait(fishDelayValue)
                end
            end)
        end
    end,
})

FarmTab:Input({
    Title = "Cast Distance",
    Value = "28", Placeholder = "28",
    Callback = function(text) local num = tonumber(text); if num then castDistValue = num end end,
})

FarmTab:Input({
    Title = "Throw Power",
    Value = "10", Placeholder = "10",
    Callback = function(text) local num = tonumber(text); if num then throwPowerValue = num end end,
})

FarmTab:Input({
    Title = "Cycle Delay (Speed)",
    Value = "0.01", Placeholder = "0.01",
    Callback = function(text) local num = tonumber(text); if num then fishDelayValue = num end end,
})

InfoTab:Select()

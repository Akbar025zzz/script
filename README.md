-- Matikan instance lama kalau ada
if _G.KingVypersRunning then
    _G.KingVypersRunning = false
end
task.wait(0.5)
_G.KingVypersRunning = true

-- â”€â”€ SNAPSHOT GUI SEBELUM VYPER LOAD â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€
-- Catat nama semua GUI yang sudah ada SEBELUM Vyper dibuat,
-- supaya kita TIDAK salah hapus GUI milik game saat re-execute.
local function snapshotGuis(container)
    local names = {}
    pcall(function()
        for _, gui in ipairs(container:GetChildren()) do
            if gui:IsA("ScreenGui") or gui:IsA("BillboardGui") or gui:IsA("SurfaceGui") then
                names[gui] = true -- track by reference, bukan nama (bisa duplikat)
            end
        end
    end)
    return names
 end

local pg = game:GetService("Players").LocalPlayer:WaitForChild("PlayerGui")
local cg = game:GetService("CoreGui")
local pgBefore = snapshotGuis(pg)
local cgBefore = snapshotGuis(cg)

-- Hapus window Vyper lama kalau ada (dari execute sebelumnya)
pcall(function()
    for _, gui in ipairs(pg:GetChildren()) do
        if gui:GetAttribute("VyperWindow") then gui:Destroy() end
    end
end)
pcall(function()
    for _, gui in ipairs(cg:GetChildren()) do
        if gui:GetAttribute("VyperWindow") then gui:Destroy() end
    end
end)

-- Tunggu game fully loaded sebelum apapun
if not game:IsLoaded() then game.Loaded:Wait() end
task.wait(1)

local Vyper = loadstring(game:HttpGet("https://gitlab.com/blazars1/blazarsui/-/raw/main/vyperuicoba.lua"))()

local Window = Vyper:Window({
    Title = "KING VYPERS",
    Footer = "|DDS PREMIUM BETA",
    Color = Color3.fromRGB(100, 200, 255),
    ["Tab Width"] = 130,
    Version = 1,
    Image = "107726435417936"
})

-- Tag HANYA GUI baru yang muncul setelah Vyper dibuat (bukan snapshot lama)
task.defer(function()
    task.wait(0.5) -- beri waktu Vyper selesai bikin semua GUI-nya
    for _, gui in ipairs(pg:GetChildren()) do
        if not pgBefore[gui] and not gui:GetAttribute("VyperWindow") then
            gui:SetAttribute("VyperWindow", true)
        end
    end
    for _, gui in ipairs(cg:GetChildren()) do
        if not cgBefore[gui] and not gui:GetAttribute("VyperWindow") then
            -- Jangan tag GUI inti Roblox
            local n = gui.Name
            if not n:find("Roblox") and not n:find("Chat") and not n:find("TopBar") and not n:find("CoreGui") then
                gui:SetAttribute("VyperWindow", true)
            end
        end
    end
end)

local Players = game:GetService("Players")
local Workspace = game:GetService("Workspace")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local RunService = game:GetService("RunService")
local LocalPlayer = Players.LocalPlayer
local Character = LocalPlayer.Character or LocalPlayer.CharacterAdded:Wait()

local UIS = game:GetService("UserInputService")
local isMobileDevice = UIS.TouchEnabled and not UIS.KeyboardEnabled

-- =============================================
-- AUTO SKIP MAIN MENU (jalan otomatis saat execute)
-- Dipakai saat rejoin: HOME -> pilih Barista -> masuk game
-- =============================================

task.spawn(function()
    local LocalPlayer = game:GetService("Players").LocalPlayer
    local playerGui = LocalPlayer:WaitForChild("PlayerGui")

    local mainMenu = playerGui:WaitForChild("mainMenuSystem", 10)
    if not mainMenu then return end
    if not mainMenu.Enabled then return end

    local mainUI = playerGui:FindFirstChild("MainUI")
    if mainUI and mainUI.Enabled then return end

    local baseFrame = mainMenu:FindFirstChild("baseFrame")
    if not baseFrame or not baseFrame.Visible then return end

    task.wait(1.5)

    local function clickBtn(btn)
        if not btn then return end
        pcall(function() firesignal(btn.MouseButton1Click) end)
        pcall(function() btn.MouseButton1Click:Fire() end)
        pcall(function() btn.Activated:Fire() end)
    end

    local applySelect = nil
    pcall(function()
        applySelect = baseFrame:FindFirstChild("homeFrame"):FindFirstChild("playFrame"):FindFirstChild("applySelect")
    end)
    print("[AutoSkip] applySelect found:", applySelect ~= nil)
    clickBtn(applySelect)
    task.wait(1.5)

    local baristaBtn = nil
    pcall(function()
        baristaBtn = baseFrame:FindFirstChild("playFrame"):FindFirstChild("ScrollingFrame"):FindFirstChild("teamFiveTeamSelect")
    end)
    print("[AutoSkip] baristaBtn found:", baristaBtn ~= nil)
    clickBtn(baristaBtn)
    task.wait(0.8)

    local deploySelect = nil
    pcall(function()
        deploySelect = baseFrame:FindFirstChild("playFrame"):FindFirstChild("deploySelect")
    end)
    print("[AutoSkip] deploySelect found:", deploySelect ~= nil)
    clickBtn(deploySelect)

    local timeout = tick() + 30
    local entered = false
    while tick() < timeout do
        local mui = playerGui:FindFirstChild("MainUI")
        if mui and mui.Enabled then
            entered = true
            break
        end
        task.wait(0.5)
    end
    print("[AutoSkip] Entered game:", entered)
    task.wait(3)

    if entered and baristaRunning == false and SpawnCar.SelectedCar and SpawnCar.SelectedCar ~= "Refresh dulu..." then
        print("[AutoSkip] Auto starting Barista loop...")
        task.spawn(startBaristaLoop)
    end
end)

-- =============================================
-- HELPER: Ambil tombol Gas dari Interface
-- =============================================

local function getGasButton()
    local interface = LocalPlayer.PlayerGui:FindFirstChild("Interface")
    if not interface then return nil end
    local buttons = interface:FindFirstChild("Buttons")
    if not buttons then return nil end
    return buttons:FindFirstChild("Gas")
end

-- =============================================
-- VEHICLE SPEED MODULE
-- =============================================

local VehicleSpeed = {}
VehicleSpeed.Enabled = false
VehicleSpeed.CurrentSpeed = 100
VehicleSpeed.BoostActive = false
VehicleSpeed.BoostLoop = nil
VehicleSpeed.DecelActive = false
VehicleSpeed.DecelLoop = nil
VehicleSpeed.InputConn1 = nil
VehicleSpeed.InputConn2 = nil
VehicleSpeed._liveSpeed = 0

local function findMyMotor()
    local myName = LocalPlayer.Name
    for _, v in pairs(Workspace:GetChildren()) do
        if v.Name:match(myName) and v.Name:match("Montors") then
            return v
        end
    end
    return nil
end

local function getMotorPrimaryPart()
    local motor = findMyMotor()
    if not motor then return nil end
    return motor.PrimaryPart or motor:FindFirstChildWhichIsA("BasePart")
end

local function isRidingMotor()
    local char = LocalPlayer.Character
    if not char then return false end
    local humanoid = char:FindFirstChild("Humanoid")
    if not humanoid then return false end
    return humanoid.Sit == true and humanoid.SeatPart ~= nil
end

-- KEY FIX: Hanya override X dan Z, biarkan Y fisika game yang handle
local function applyVelocity(p, dir, targetSpeed, blend)
    local vel = p.AssemblyLinearVelocity
    local newX = vel.X + (dir.X * targetSpeed - vel.X) * blend
    local newZ = vel.Z + (dir.Z * targetSpeed - vel.Z) * blend
    -- Y TIDAK diubah sama sekali â€” biarkan physics engine yang handle gravity & suspensi
    p.AssemblyLinearVelocity = Vector3.new(newX, vel.Y, newZ)
end

local function stopDecelLoop()
    VehicleSpeed.DecelActive = false
    if VehicleSpeed.DecelLoop then
        task.cancel(VehicleSpeed.DecelLoop)
        VehicleSpeed.DecelLoop = nil
    end
end

local function startBoostLoop()
    stopDecelLoop()
    if VehicleSpeed.BoostActive then return end
    VehicleSpeed.BoostActive = true
    VehicleSpeed.BoostLoop = task.spawn(function()
        local currentSpeed = VehicleSpeed._liveSpeed
        local p = getMotorPrimaryPart()
        if p then
            local vel = p.AssemblyLinearVelocity
            local flatMag = Vector3.new(vel.X, 0, vel.Z).Magnitude
            currentSpeed = math.max(currentSpeed, flatMag)
        end

        while VehicleSpeed.BoostActive and VehicleSpeed.Enabled do
            if isRidingMotor() then
                p = getMotorPrimaryPart()
                if p then
                    local dir = -p.CFrame.LookVector
                    currentSpeed = currentSpeed + (VehicleSpeed.CurrentSpeed - currentSpeed) * 0.2
                    VehicleSpeed._liveSpeed = currentSpeed
                    applyVelocity(p, dir, currentSpeed, 0.35)
                end
            end
            task.wait(0.05)
        end
    end)
end

local function stopBoostLoop()
    VehicleSpeed.BoostActive = false
    if VehicleSpeed.BoostLoop then
        task.cancel(VehicleSpeed.BoostLoop)
        VehicleSpeed.BoostLoop = nil
    end
    VehicleSpeed._liveSpeed = 0
end

function VehicleSpeed.Start()
    if VehicleSpeed.Enabled then return end
    VehicleSpeed.Enabled = true

    if isMobileDevice then
        task.spawn(function()
            local gasBtn = nil
            while VehicleSpeed.Enabled and not gasBtn do
                gasBtn = getGasButton()
                if not gasBtn then task.wait(0.5) end
            end
            if not gasBtn then return end

            VehicleSpeed.InputConn1 = gasBtn.MouseButton1Down:Connect(function()
                if VehicleSpeed.Enabled then startBoostLoop() end
            end)
            VehicleSpeed.InputConn2 = gasBtn.MouseButton1Up:Connect(function()
                if VehicleSpeed.Enabled then stopBoostLoop() end
            end)
        end)
    else
        VehicleSpeed.InputConn1 = UIS.InputBegan:Connect(function(input, gpe)
            if gpe then return end
            if not VehicleSpeed.Enabled then return end
            if input.KeyCode == Enum.KeyCode.W or input.KeyCode == Enum.KeyCode.Up then
                startBoostLoop()
            end
        end)
        VehicleSpeed.InputConn2 = UIS.InputEnded:Connect(function(input, gpe)
            if not VehicleSpeed.Enabled then return end
            if input.KeyCode == Enum.KeyCode.W or input.KeyCode == Enum.KeyCode.Up then
                if not UIS:IsKeyDown(Enum.KeyCode.W) and not UIS:IsKeyDown(Enum.KeyCode.Up) then
                    stopBoostLoop()
                end
            end
        end)
    end
end

function VehicleSpeed.Stop()
    if not VehicleSpeed.Enabled then return end
    VehicleSpeed.Enabled = false
    VehicleSpeed.BoostActive = false
    if VehicleSpeed.BoostLoop then task.cancel(VehicleSpeed.BoostLoop) VehicleSpeed.BoostLoop = nil end
    VehicleSpeed._liveSpeed = 0
    if VehicleSpeed.InputConn1 then VehicleSpeed.InputConn1:Disconnect() VehicleSpeed.InputConn1 = nil end
    if VehicleSpeed.InputConn2 then VehicleSpeed.InputConn2:Disconnect() VehicleSpeed.InputConn2 = nil end
end

function VehicleSpeed.SetSpeed(speed)
    VehicleSpeed.CurrentSpeed = speed
end

-- =============================================
-- SLOW RACE MODULE (Gradual Acceleration)
-- =============================================

local SlowRace = {}
SlowRace.Enabled = false
SlowRace.MaxSpeed = 200
SlowRace.AccelMultiplier = 3
SlowRace.BoostActive = false
SlowRace.BoostLoop = nil
SlowRace.DecelActive = false
SlowRace.DecelLoop = nil
SlowRace.InputConn1 = nil
SlowRace.InputConn2 = nil
SlowRace._liveSpeed = 0

local function stopSlowDecelLoop()
    SlowRace.DecelActive = false
    if SlowRace.DecelLoop then
        task.cancel(SlowRace.DecelLoop)
        SlowRace.DecelLoop = nil
    end
end

local function startSlowRaceLoop()
    stopSlowDecelLoop()
    if SlowRace.BoostActive then return end
    SlowRace.BoostActive = true
    SlowRace.BoostLoop = task.spawn(function()
        local currentSpeed = SlowRace._liveSpeed
        local p = getMotorPrimaryPart()
        if p then
            local vel = p.AssemblyLinearVelocity
            local flatMag = Vector3.new(vel.X, 0, vel.Z).Magnitude
            currentSpeed = math.max(currentSpeed, flatMag)
        end

        while SlowRace.BoostActive and SlowRace.Enabled do
            if isRidingMotor() then
                p = getMotorPrimaryPart()
                if p then
                    currentSpeed = math.min(
                        currentSpeed + (SlowRace.AccelMultiplier * 0.5),
                        SlowRace.MaxSpeed
                    )
                    SlowRace._liveSpeed = currentSpeed
                    local dir = -p.CFrame.LookVector
                    applyVelocity(p, dir, currentSpeed, 0.3)
                end
            end
            task.wait(0.05)
        end
    end)
end

local function stopSlowRaceLoop()
    SlowRace.BoostActive = false
    if SlowRace.BoostLoop then
        task.cancel(SlowRace.BoostLoop)
        SlowRace.BoostLoop = nil
    end
    SlowRace._liveSpeed = 0
end

function SlowRace.Start()
    if SlowRace.Enabled then return end
    SlowRace.Enabled = true

    if isMobileDevice then
        task.spawn(function()
            local gasBtn = nil
            while SlowRace.Enabled and not gasBtn do
                gasBtn = getGasButton()
                if not gasBtn then task.wait(0.5) end
            end
            if not gasBtn then return end

            SlowRace.InputConn1 = gasBtn.MouseButton1Down:Connect(function()
                if SlowRace.Enabled then startSlowRaceLoop() end
            end)
            SlowRace.InputConn2 = gasBtn.MouseButton1Up:Connect(function()
                if SlowRace.Enabled then stopSlowRaceLoop() end
            end)
        end)
    else
        SlowRace.InputConn1 = UIS.InputBegan:Connect(function(input, gpe)
            if gpe then return end
            if not SlowRace.Enabled then return end
            if input.KeyCode == Enum.KeyCode.W or input.KeyCode == Enum.KeyCode.Up then
                startSlowRaceLoop()
            end
        end)
        SlowRace.InputConn2 = UIS.InputEnded:Connect(function(input, gpe)
            if not SlowRace.Enabled then return end
            if input.KeyCode == Enum.KeyCode.W or input.KeyCode == Enum.KeyCode.Up then
                if not UIS:IsKeyDown(Enum.KeyCode.W) and not UIS:IsKeyDown(Enum.KeyCode.Up) then
                    stopSlowRaceLoop()
                end
            end
        end)
    end
end

function SlowRace.Stop()
    if not SlowRace.Enabled then return end
    SlowRace.Enabled = false
    SlowRace.BoostActive = false
    if SlowRace.BoostLoop then task.cancel(SlowRace.BoostLoop) SlowRace.BoostLoop = nil end
    SlowRace._liveSpeed = 0
    if SlowRace.InputConn1 then SlowRace.InputConn1:Disconnect() SlowRace.InputConn1 = nil end
    if SlowRace.InputConn2 then SlowRace.InputConn2:Disconnect() SlowRace.InputConn2 = nil end
end

-- =============================================
-- SPAWN CAR MODULE
-- =============================================

local SpawnCar = {}
SpawnCar.SelectedCar = nil
SpawnCar.CarList = {}
SpawnCar.AutoRide = false
SpawnCar.AutoRideAlways = false

local function exitMotor()
    local motor = findMyMotor()
    if not motor then return false end
    local char = LocalPlayer.Character
    if not char then return false end

    local anims = motor:FindFirstChild("Anims")
    if anims then
        pcall(function() anims:FireServer("RemovePlayer", char, nil) end)
        task.wait(0.3)
    end

    local driveSeat = motor:FindFirstChild("DriveSeat", true)
    if driveSeat then
        pcall(function() driveSeat:Sit(nil) end)
        task.wait(0.3)
    end

    local humanoid = char:FindFirstChildOfClass("Humanoid")
    if humanoid then
        pcall(function() humanoid.Jump = true end)
    end

    return true
end

local function rideMotor()
    local motor = findMyMotor()
    if not motor then return false end

    local char = LocalPlayer.Character
    if not char then return false end

    local anims = motor:FindFirstChild("Anims")
    if anims then
        pcall(function() anims:FireServer("CreatePlayer", char) end)
        task.wait(0.2)
        pcall(function() anims:FireServer("RegisterPlayer", char) end)
        task.wait(0.2)
    end

    local kickstand = motor:FindFirstChild("Kickstand")
    if kickstand then
        pcall(function() kickstand:FireServer("StandUp", 0, 0, 0, 0, false) end)
        task.wait(0.2)
    end

    local driveSeat = motor:FindFirstChild("DriveSeat", true)
    if driveSeat then
        pcall(function()
            local hrp = char:FindFirstChild("HumanoidRootPart")
            if hrp then hrp.CFrame = driveSeat.CFrame end
            driveSeat:Sit(char:FindFirstChildOfClass("Humanoid"))
        end)
    end
    return true
end

local function getCarList()
    local carNames = {}
    pcall(function()
        local mainUI = LocalPlayer.PlayerGui:WaitForChild("MainUI")
        local frame = mainUI:WaitForChild("Frame")
        local spawnBtn = mainUI:WaitForChild("Spawn"):WaitForChild("SpawnCar")
        frame.Visible = false
        firesignal(spawnBtn.MouseButton1Click)
        task.wait(2)
        local sf = frame:WaitForChild("MainFrame"):WaitForChild("ScrollingFrame")
        for _, v in ipairs(sf:GetChildren()) do
            if v.ClassName:sub(1,2) ~= "UI" then
                table.insert(carNames, v.Name)
            end
        end
        frame.Visible = false
    end)
    SpawnCar.CarList = carNames
    return #carNames > 0 and carNames or { "Refresh dulu..." }
end

function SpawnCar.Spawn()
    if not SpawnCar.SelectedCar or SpawnCar.SelectedCar == "Refresh dulu..." then return end
    local ok = pcall(function()
        ReplicatedStorage:WaitForChild("SpawnCarEvents"):WaitForChild("SpawnCar"):FireServer(SpawnCar.SelectedCar)
    end)
    if ok and SpawnCar.AutoRide then
        task.spawn(function()
            task.wait(4)
            rideMotor()
        end)
    end
end

function SpawnCar.Despawn()
    pcall(function()
        ReplicatedStorage:WaitForChild("SpawnCarEvents"):WaitForChild("DespawnCar"):FireServer()
    end)
end

local autoRideLoop = nil
function SpawnCar.StartAutoRideAlways()
    if autoRideLoop then return end
    autoRideLoop = task.spawn(function()
        while SpawnCar.AutoRideAlways do
            local humanoid = LocalPlayer.Character and LocalPlayer.Character:FindFirstChildOfClass("Humanoid")
            if humanoid and not humanoid.Sit and findMyMotor() then
                rideMotor()
            end
            task.wait(2)
        end
        autoRideLoop = nil
    end)
end

function SpawnCar.StopAutoRideAlways()
    SpawnCar.AutoRideAlways = false
    if autoRideLoop then task.cancel(autoRideLoop) autoRideLoop = nil end
end

-- =============================================
-- AUTO JOB BARISTA MODULE
-- =============================================

local BaristaJob = { Name = "Barista", TeamId = 11378976, X = -5022.51, Y = 3.18, Z = -676.96 }

local function setJob(job)
    pcall(function()
        ReplicatedStorage:WaitForChild("JobEvents"):WaitForChild("TeamChangeRequest")
            :FireServer(job.Name, job.TeamId, 1, 0, "Detector")
    end)
end

local function walkTo(humanoid, point, timeout)
    timeout = timeout or 10
    local t = tick()
    while tick() - t < timeout do
        local hrp = LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart")
        if hrp and (hrp.Position - point).Magnitude < 5 then break end
        humanoid:MoveTo(point)
        task.wait(0.5)
    end
end

local function getBaristaEvent()
    local assets = ReplicatedStorage:FindFirstChild("BaristaAssets")
    if not assets then return nil end
    local events = assets:FindFirstChild("Events")
    if not events then return nil end
    return events:FindFirstChild("BaristaEvent")
end

-- =============================================
-- AUTO COMPLETE MINIGAME - HUMAN-LIKE ANTI DETEKSI
-- =============================================

local PlayerGui = LocalPlayer:WaitForChild("PlayerGui")

-- task.spawn(function()
--     math.randomseed(os.clock() * 1000)

--     local playerGui = LocalPlayer:WaitForChild("PlayerGui")

--     local baristaGui = playerGui:FindFirstChild("BaristaGUI")
--     while not baristaGui do
--         task.wait(1)
--         baristaGui = playerGui:FindFirstChild("BaristaGUI")
--     end

--     local minigame     = baristaGui:WaitForChild("MinigameFrame")
--     local bgBar        = minigame:WaitForChild("BackgroundBar")
--     local tapZone      = minigame:WaitForChild("TapZone")
--     local targetZone   = bgBar:WaitForChild("TargetZone")
--     local playerCursor = bgBar:WaitForChild("PlayerCursor")

--     local function tap()
--         local r = math.random(1, 3)
--         if r == 1 then
--             pcall(function() firesignal(tapZone.MouseButton1Click) end)
--         elseif r == 2 then
--             pcall(function() firesignal(tapZone.MouseButton1Down) end)
--             task.wait(math.random(5, 15) * 0.01)
--             pcall(function() firesignal(tapZone.MouseButton1Up) end)
--         else
--             pcall(function() firesignal(tapZone.MouseButton1Click) end)
--         end
--     end

--     local lastTapTime = 0

--     while true do
--         task.wait(math.random(10, 25) * 0.001)

--         if not minigame.Visible then
--             task.wait(0.2)
--             continue
--         end

--         local targetMid = targetZone.AbsolutePosition.Y + targetZone.AbsoluteSize.Y / 2
--         local cursorMid = playerCursor.AbsolutePosition.Y + playerCursor.AbsoluteSize.Y / 2
--         local diff = cursorMid - targetMid
--         local now = tick()

--         if diff > 10 then
--             -- Bar putih di BAWAH kapsul kuning â†’ tap buat naik
--             local urgency = math.clamp(diff / 80, 0.3, 1.0)
--             local interval = math.random(30, 70) * 0.001 / urgency
--             if now - lastTapTime >= interval then
--                 tap()
--                 lastTapTime = now
--             end
--         elseif diff > -10 then
--             -- Sudah sejajar â†’ tap pelan buat maintain
--             if math.random(1, 100) <= 20 then
--                 local interval = math.random(80, 150) * 0.001
--                 if now - lastTapTime >= interval then
--                     tap()
--                     lastTapTime = now
--                 end
--             end
--         end
--         -- diff < -10 = cursor di atas â†’ jangan tap, tunggu turun sendiri
--     end
-- end)

task.spawn(function()
    math.randomseed(os.clock() * 1000)

    local playerGui = LocalPlayer:WaitForChild("PlayerGui")

    local baristaGui = playerGui:FindFirstChild("BaristaGUI")
    while not baristaGui do
        task.wait(1)
        baristaGui = playerGui:FindFirstChild("BaristaGUI")
    end

    local minigame     = baristaGui:WaitForChild("MinigameFrame")
    local bgBar        = minigame:WaitForChild("BackgroundBar")
    local tapZone      = minigame:WaitForChild("TapZone")
    local targetZone   = bgBar:WaitForChild("TargetZone")
    local playerCursor = bgBar:WaitForChild("PlayerCursor")

    local function tap()
        local r = math.random(1, 2)
        if r == 1 then
            pcall(function() firesignal(tapZone.MouseButton1Click) end)
        else
            pcall(function() firesignal(tapZone.MouseButton1Down) end)
            task.wait(math.random(3, 8) * 0.01)
            pcall(function() firesignal(tapZone.MouseButton1Up) end)
        end
    end

    local lastTapTime = 0

    while true do
        task.wait(math.random(10, 20) * 0.001)

        if not minigame.Visible then
            task.wait(0.2)
            continue
        end

        local targetPos  = targetZone.AbsolutePosition.Y
        local targetSize = targetZone.AbsoluteSize.Y
        local targetTop  = targetPos
        local targetBot  = targetPos + targetSize

        local cursorMid = playerCursor.AbsolutePosition.Y + playerCursor.AbsoluteSize.Y / 2
        local now = tick()

        if cursorMid > targetBot then
            -- Cursor di bawah target â†’ tap lebih sering buat naik
            local diff = cursorMid - targetBot
            local interval = math.max(0.02, 0.08 - (diff / 500))
            if now - lastTapTime >= interval then
                tap()
                lastTapTime = tick()
            end
        elseif cursorMid >= targetTop and cursorMid <= targetBot then
            -- Cursor di dalam target â†’ tap pelan maintain
            if math.random(1, 100) <= 30 then
                local interval = math.random(60, 100) * 0.001
                if now - lastTapTime >= interval then
                    tap()
                    lastTapTime = tick()
                end
            end
        end
        -- Cursor di atas target â†’ jangan tap sama sekali
    end
end)

-- =============================================
-- FUNGSI SERVE ORDER & TRY BREW
-- =============================================

local function doTryBrew()
    local ok, prompt = pcall(function()
        return Workspace.BaristaJob.Interactions.MachinePart.MachinePart.MachinePrompt
    end)
    if not ok or not prompt then
        print("[doTryBrew] MachinePrompt tidak ditemukan!")
        return false
    end
    print("[doTryBrew] Hold MachinePrompt selama:", prompt.HoldDuration)
    prompt:InputHoldBegin()
    task.wait(prompt.HoldDuration + 0.1)
    prompt:InputHoldEnd()
    return true
end

local function doServeOrder()
    pcall(function()
        local prompt = Workspace.BaristaJob.Interactions.RegisterPart.RegisterPart.RegisterPrompt
        -- Tunggu prompt enabled dulu (max 10 detik)
        local t = tick()
        while not prompt.Enabled and tick() - t < 10 do
            task.wait(0.2)
        end
        if not prompt.Enabled then return end
        prompt:InputHoldBegin()
        task.wait(prompt.HoldDuration + 0.05)
        prompt:InputHoldEnd()
    end)
end

-- =============================================
-- FIX MACHINE (Repair Brewing Machine)
-- Dipanggil kalau MachinePrompt.ActionText berubah = mesin rusak
-- =============================================


-- Waypoint route ke mesin dan balik
local MACHINE_ROUTE = {
    Vector3.new(-4995.341309, 4.289086, -801.850220), -- Pos 1
    Vector3.new(-5007.872070, 4.289086, -801.314514), -- Pos 2
    Vector3.new(-5006.809082, 4.289086, -728.798645), -- Pos 3
    Vector3.new(-4999.592773, 3.189320, -670.799133), -- Pos 4
    Vector3.new(-5113.675781, 3.189320, -672.781311), -- Pos 5 (mesin)
}

local fixingMachine = false -- flag: sedang fix mesin, pause loop utama
local machineNeedsRepair = false -- dipindah ke sini agar bisa diakses oleh doFixMachine

local function doFixMachine()
    fixingMachine = true
    -- machineNeedsRepair di-reset SETELAH UI confirm beres, bukan di sini

    local humanoid = LocalPlayer.Character and LocalPlayer.Character:FindFirstChildOfClass("Humanoid")
    if not humanoid then fixingMachine = false return end

    -- Jalan ke mesin lewat waypoint 1â†’2â†’3â†’4â†’5
    for _, pos in ipairs(MACHINE_ROUTE) do
        humanoid = LocalPlayer.Character and LocalPlayer.Character:FindFirstChildOfClass("Humanoid")
        if not humanoid then fixingMachine = false return end
        walkTo(humanoid, pos, 15)
    end

    task.wait(0.5)

    -- Cari SupplyPrompt (bukan MachinePrompt!)
    -- Path: Workspace.BaristaJob.Interactions.SupplyPart.SupplyPart.SupplyPrompt
    local supplyPrompt = nil
    pcall(function()
        supplyPrompt = Workspace.BaristaJob.Interactions.SupplyPart.SupplyPart.SupplyPrompt
    end)

    -- Fallback: scan workspace kalau path berubah
    if not supplyPrompt then
        for _, v in pairs(Workspace:GetDescendants()) do
            if v.Name == "SupplyPrompt" and v:IsA("ProximityPrompt") then
                supplyPrompt = v
                break
            end
        end
    end

    print("[FixMachine] SupplyPrompt found:", supplyPrompt ~= nil)

    if supplyPrompt then
        -- Hold pakai HoldDuration dari prompt itu sendiri
        pcall(function()
            supplyPrompt:InputHoldBegin()
            task.wait(supplyPrompt.HoldDuration + 0.05)
            supplyPrompt:InputHoldEnd()
        end)
    end

    task.wait(1)

    -- Tunggu UI confirm mesin sudah beres (max 15 detik)
    -- Kalau ObjectiveLabel dan OrderText sudah tidak "Fix Machine" / "Machine broke down!" = selesai
    local waitTimeout = tick() + 15
    while tick() < waitTimeout do
        local obj1 = ""
        local obj2 = ""
        pcall(function()
            obj1 = PlayerGui.BaristaMissionUI.Container.MainFrame.Frame.ObjectiveLabel.Text
        end)
        pcall(function()
            obj2 = PlayerGui.BaristaGUI.StatusFrame.OrderText.Text
        end)
        if obj1 ~= "Fix Machine" and obj2 ~= "Machine broke down!" then
            print("[FixMachine] Mesin sudah beres! Lanjut balik...")
            break
        end
        task.wait(0.5)
    end

    -- Reset flag SETELAH UI confirm beres
    machineNeedsRepair = false
    task.wait(1)

    -- Balik lewat waypoint terbalik: 4â†’3â†’2â†’1
    humanoid = LocalPlayer.Character and LocalPlayer.Character:FindFirstChildOfClass("Humanoid")
    if humanoid then
        for i = #MACHINE_ROUTE - 1, 1, -1 do
            humanoid = LocalPlayer.Character and LocalPlayer.Character:FindFirstChildOfClass("Humanoid")
            if not humanoid then break end
            walkTo(humanoid, MACHINE_ROUTE[i], 15)
        end
    end

    task.wait(0.5)
    fixingMachine = false
end



-- =============================================
-- AUTO BARISTA GOTO + LOOP
-- =============================================

local baristaRunning = false

-- Monitor mesin rusak: cek ObjectiveLabel + OrderText setiap 1 detik
-- var machineNeedsRepair dipindah ke atas

task.spawn(function()
    -- Tunggu sampai barista loop jalan dulu
    while not baristaRunning do task.wait(1) end
    print("[MachineMonitor] Mulai monitor mesin...")

    while baristaRunning and _G.KingVypersRunning do
        task.wait(1)

        if fixingMachine then continue end

        local obj1 = ""
        local obj2 = ""

        pcall(function()
            obj1 = PlayerGui.BaristaMissionUI.Container.MainFrame.Frame.ObjectiveLabel.Text
        end)
        pcall(function()
            obj2 = PlayerGui.BaristaGUI.StatusFrame.OrderText.Text
        end)

        if obj1 == "Fix Machine" or obj2 == "Machine broke down!" then
            if not machineNeedsRepair then
                print("[MachineMonitor] MESIN RUSAK! obj1=[" .. obj1 .. "] obj2=[" .. obj2 .. "]")
                machineNeedsRepair = true
            end
        end
    end

    print("[MachineMonitor] Monitor berhenti")
end)

local function startBaristaLoop()
    local job = BaristaJob

    if baristaRunning then return end
    if not SpawnCar.SelectedCar or SpawnCar.SelectedCar == "Refresh dulu..." then return end
    baristaRunning = true

    setJob(job)
    task.wait(0.5)

    pcall(function()
        ReplicatedStorage:WaitForChild("SpawnCarEvents"):WaitForChild("SpawnCar"):FireServer(SpawnCar.SelectedCar)
    end)

    task.wait(4)
    rideMotor()
    task.wait(2)

    local char = LocalPlayer.Character
    local hrp = char and char:FindFirstChild("HumanoidRootPart")
    local motor = findMyMotor()

    if not (motor and hrp) then baristaRunning = false return end

    local target = CFrame.new(job.X, job.Y, job.Z)
    pcall(function()
        motor:SetPrimaryPartCFrame(target)
        task.wait(0.1)
        hrp.CFrame = target * CFrame.new(0, 2, 0)
    end)

    task.wait(2)
    exitMotor()
    task.wait(0.5)

    local humanoid = LocalPlayer.Character and LocalPlayer.Character:FindFirstChildOfClass("Humanoid")
    if not humanoid then baristaRunning = false return end

    local approachPoints = {
        Vector3.new(-5004.69, 3.18, -675.35),
        Vector3.new(-5003.38, 4.28, -715.09),
        Vector3.new(-4991.12, 4.28, -714.90),
    }
    for _, pos in ipairs(approachPoints) do
        walkTo(humanoid, pos, 10)
    end

    task.wait(0.5)
    -- Start job pakai JobPrompt
    pcall(function()
        local prompt = Workspace.BaristaJob.Interactions.StartPart.StartPart.JobPrompt
        prompt:InputHoldBegin()
        task.wait(prompt.HoldDuration + 0.05)
        prompt:InputHoldEnd()
    end)
    task.wait(0.5)

    local innerPoints = {
        Vector3.new(-5007.13, 4.28, -757.45),
        Vector3.new(-5006.95, 4.28, -801.65),
        Vector3.new(-4996.64, 4.28, -801.13),
        Vector3.new(-4997.60, 4.28, -794.40),
    }
    for _, pos in ipairs(innerPoints) do
        walkTo(humanoid, pos, 15)
    end

    while baristaRunning and _G.KingVypersRunning do
        humanoid = LocalPlayer.Character and LocalPlayer.Character:FindFirstChildOfClass("Humanoid")
        if not humanoid then task.wait(2) continue end

        -- PRIORITAS: kalau mesin rusak, pause semua dan fix dulu
        if machineNeedsRepair and not fixingMachine then
            print("[BaristaLoop] Mesin rusak terdeteksi! Memulai doFixMachine...")
            task.spawn(doFixMachine)
        end

        -- Tunggu sampai fix selesai sebelum lanjut brew
        if fixingMachine then
            task.wait(0.5)
            continue
        end

        -- Step 1: Jalan ke posisi mesin brewing
        walkTo(humanoid, Vector3.new(-4997.60, 4.28, -794.40), 10)
        task.wait(0.5)

        -- Step 2: Hold MachinePrompt (trigger brew + minigame)
        local brewOk = doTryBrew()

        if not brewOk then
            -- Prompt gagal, tunggu sebentar lalu retry dari awal loop
            print("[BaristaLoop] doTryBrew gagal, retry...")
            task.wait(2)
            continue
        end

        -- Step 3: Tunggu minigame muncul (max 10 detik)
        local bGui = PlayerGui:FindFirstChild("BaristaGUI")
        local mgAppeared = false
        if bGui then
            local t = tick()
            while tick() - t < 10 do
                local mf = bGui:FindFirstChild("MinigameFrame")
                if mf and mf.Visible then
                    mgAppeared = true
                    break
                end
                task.wait(0.2)
            end
        end

        -- Step 4: Tunggu minigame selesai (max 30 detik)
        if mgAppeared then
            local timeout = tick() + 30
            local mf = bGui and bGui:FindFirstChild("MinigameFrame")
            while mf and mf.Visible and tick() < timeout do
                task.wait(0.2)
            end
        end

        task.wait(1)

        -- Step 5: Jalan ke kasir dan serve order
        humanoid = LocalPlayer.Character and LocalPlayer.Character:FindFirstChildOfClass("Humanoid")
        if humanoid then
            walkTo(humanoid, Vector3.new(-4995.49, 4.28, -761.86), 15)
        end
        task.wait(0.5)
        doServeOrder()

        task.wait(5)
    end
end

-- =============================================
-- AUTO JOB COURIER MODULE
-- =============================================
-- =============================================
-- AUTO JOB COURIER MODULE
-- =============================================
local CourierJob = { Name = "Courier", TeamId = 11378976, X = -5158.57, Y = 4.41, Z = -3757.87 }
local courierRunning = false
local ServiceEventConn = nil

local function startCourierLoop()
    local job = CourierJob

    if courierRunning then return end
    if not SpawnCar.SelectedCar or SpawnCar.SelectedCar == "Refresh dulu..." then return end
    courierRunning = true

    local activePackageLoc = nil
    local activePackageNum = nil
    
    local serviceEvent = ReplicatedStorage:FindFirstChild("ServiceEvent", true)
    if serviceEvent then
        ServiceEventConn = serviceEvent.OnClientEvent:Connect(function(eventName, action, paketNum)
            if action == "Create" then
                local Location = workspace:FindFirstChild("Livrason") and workspace.Livrason:FindFirstChild("Location")
                if Location then
                    local paket = Location:FindFirstChild(tostring(paketNum))
                    if paket then
                        local point = paket:FindFirstChild("POINT")
                        local block = paket:FindFirstChild("Block")
                        local billboard = point and point:FindFirstChild("billboardgui")
                        local namaLokasi = ""
                        if billboard then
                            for _, gui in ipairs(billboard:GetChildren()) do
                                if gui:IsA("TextLabel") and gui.Text ~= "ðŸ”»" and gui.Text ~= "" then
                                    namaLokasi = gui.Text
                                end
                            end
                        end
                        if point and block then
                            activePackageLoc = block.Position
                            activePackageNum = paketNum
                            print("=== AMBIL PAKET ===")
                            print("Paket #" .. tostring(paketNum))
                            print("Lokasi: " .. namaLokasi)
                            print("Position: " .. tostring(activePackageLoc))
                        end
                    end
                end
            elseif action == "Remove" then
                print("Paket #" .. tostring(paketNum) .. " selesai!")
                if activePackageNum == paketNum then
                    activePackageLoc = nil
                    activePackageNum = nil
                end
            end
        end)
    end

    setJob(job)
    task.wait(1.5)

    pcall(function()
        ReplicatedStorage:WaitForChild("SpawnCarEvents"):WaitForChild("SpawnCar"):FireServer(SpawnCar.SelectedCar)
    end)

    task.wait(6)
    rideMotor()
    task.wait(3.5)

    local char = LocalPlayer.Character
    local hrp = char and char:FindFirstChild("HumanoidRootPart")
    local motor = findMyMotor()

    if not (motor and hrp) then courierRunning = false return end

    local target = CFrame.new(job.X, job.Y, job.Z)
    pcall(function()
        motor:SetPrimaryPartCFrame(target)
        task.wait(0.3)
        hrp.CFrame = target * CFrame.new(0, 2, 0)
    end)

    task.wait(3.5)
    exitMotor()
    task.wait(1.5)

    local humanoid = LocalPlayer.Character and LocalPlayer.Character:FindFirstChildOfClass("Humanoid")
    if not humanoid then courierRunning = false return end

    walkTo(humanoid, Vector3.new(-5109.06, 5.18, -3758.69), 10)
    task.wait(1.5)

    -- Ambil Paket di pickup point
    pcall(function()
        local prompt = workspace.Livrason.Take1.Take.ProximityPrompt
        if prompt then
            prompt:InputHoldBegin()
            task.wait(prompt.HoldDuration + 0.2)
            prompt:InputHoldEnd()
        end
    end)
    task.wait(1.5)

    -- =========================================================
    -- GHOST GLIDE: nembus tembok langsung ke paket
    -- =========================================================
    local function forceDismount()
        local char = LocalPlayer.Character
        local hum = char and char:FindFirstChild("Humanoid")
        if not char or not hum then return end
        print("Turun dari motor...")
        hum.Sit = false
        hum.Jump = true
        task.wait(0.1)
        if hum.SeatPart then
            char:PivotTo(char:GetPivot() * CFrame.new(0, 2, 0))
            hum.Sit = false
            hum.Jump = true
        end
        task.wait(0.2)
    end

    local function ghostGlideMotor(targetPos)
        local char = LocalPlayer.Character
        local hum = char and char:FindFirstChild("Humanoid")
        local seat = hum and hum.SeatPart
        local vehicle = seat and seat:FindFirstAncestorOfClass("Model")
        if not (vehicle and vehicle.PrimaryPart) then return end

        local pp = vehicle.PrimaryPart
        local speed = 150
        local glideHeight = targetPos.Y + 3
        local posTujuan = Vector3.new(targetPos.X, glideHeight, targetPos.Z)

        print("Mengaktifkan Ghost Rider mode...")

        local virtualAnchor = Instance.new("BodyVelocity")
        virtualAnchor.MaxForce = Vector3.new(math.huge, math.huge, math.huge)
        virtualAnchor.Velocity = Vector3.new(0, 0, 0)
        virtualAnchor.Parent = pp

        local virtualGyro = Instance.new("BodyGyro")
        virtualGyro.MaxTorque = Vector3.new(math.huge, math.huge, math.huge)
        virtualGyro.P = 100000
        virtualGyro.Parent = pp

        local noclip = RunService.Stepped:Connect(function()
            for _, v in pairs(vehicle:GetDescendants()) do
                if v:IsA("BasePart") then v.CanCollide = false end
            end
            for _, v in pairs(char:GetDescendants()) do
                if v:IsA("BasePart") then v.CanCollide = false end
            end
        end)

        local _, currentYRot, _ = pp.CFrame:ToEulerAnglesYXZ()

        local function glideTo(targetVector, faceForward)
            local dist = (pp.Position - targetVector).Magnitude
            local timeToMove = dist / speed
            if timeToMove > 0 then
                local startTime = tick()
                while tick() - startTime < timeToMove do
                    if not hum.SeatPart then break end
                    local alpha = (tick() - startTime) / timeToMove
                    local currentPos = pp.Position:Lerp(targetVector, alpha)
                    if faceForward then
                        local dir = (targetVector - pp.Position).Unit
                        local flatDir = Vector3.new(dir.X, 0, dir.Z).Unit
                        if flatDir.Magnitude > 0.001 then
                            local newCFrame = CFrame.lookAt(currentPos, currentPos + flatDir)
                            virtualGyro.CFrame = newCFrame
                            vehicle:PivotTo(newCFrame)
                        end
                    else
                        local newCFrame = CFrame.new(currentPos) * CFrame.Angles(0, currentYRot, 0)
                        virtualGyro.CFrame = newCFrame
                        vehicle:PivotTo(newCFrame)
                    end
                    RunService.Heartbeat:Wait()
                end
            end
        end

        print("Terbang menembus tembok ke target...")
        glideTo(posTujuan, true)

        print("Cari aspal untuk mendarat...")
        local finalSafeY = targetPos.Y + 3
        local timeout = tick() + 8
        while tick() < timeout do
            local rayOrigin = Vector3.new(targetPos.X, glideHeight + 5, targetPos.Z)
            local rayResult = workspace:Raycast(rayOrigin, Vector3.new(0, -100, 0))
            if rayResult and rayResult.Instance then
                finalSafeY = rayResult.Position.Y + 1.5
                print("Aspal ketemu! Mendarat...")
                break
            else
                task.wait(1)
            end
        end

        glideTo(Vector3.new(targetPos.X, finalSafeY, targetPos.Z), false)

        virtualAnchor:Destroy()
        virtualGyro:Destroy()
        noclip:Disconnect()
        pp.AssemblyLinearVelocity = Vector3.new(0, 0, 0)
        pp.AssemblyAngularVelocity = Vector3.new(0, 0, 0)

        forceDismount()
    end
    -- =========================================================

    -- Loop terus antar paket selama ada yang masuk
    while courierRunning do
        -- Tunggu paket baru (max 20 detik), kalau kosong stop
        local t = tick()
        while courierRunning and not activePackageLoc and tick() - t < 20 do
            task.wait(0.4)
        end

        if not activePackageLoc then
            print("Tidak ada paket baru dalam 20 detik. Auto Courier berhenti.")
            break
        end

        local curChar = LocalPlayer.Character
        local curHrp = curChar and curChar:FindFirstChild("HumanoidRootPart")
        local curHum = curChar and curChar:FindFirstChildOfClass("Humanoid")

        -- Spawn motor baru, naik, lalu ghost glide
        print("Spawn motor...")
        pcall(function()
            ReplicatedStorage:WaitForChild("SpawnCarEvents"):WaitForChild("SpawnCar"):FireServer(SpawnCar.SelectedCar)
        end)
        task.wait(4)
        rideMotor()
        task.wait(3.5)

        print("Ghost Glide ke lokasi paket #" .. tostring(activePackageNum) .. "...")
        ghostGlideMotor(activePackageLoc)
        task.wait(1)

        -- Refresh char setelah dismount
        curChar = LocalPlayer.Character
        curHum = curChar and curChar:FindFirstChildOfClass("Humanoid")

        -- Jalan kaki ke paket
        print("Berjalan kaki ke paket...")
        local walkHum = LocalPlayer.Character and LocalPlayer.Character:FindFirstChildOfClass("Humanoid")
        if walkHum then
            walkTo(walkHum, activePackageLoc, 20)
            task.wait(2.0)
        end

        print("Sampai! Mengambil/Menaruh paket...")

        -- Simpan num sekarang sebelum di-nil oleh event Remove
        local targetNum = activePackageNum

        pcall(function()
            local LocationFolder = workspace.Livrason.Location
            local paketModel = LocationFolder:FindFirstChild(tostring(targetNum))
            if paketModel then
                local block = paketModel:FindFirstChild("Block")
                local prompt = block and block:FindFirstChild("ProximityPrompt")
                if prompt and prompt.Enabled then
                    print("Equip box & taruh paket #" .. tostring(targetNum))
                    local box = LocalPlayer.Backpack:FindFirstChild("Box")
                        or curChar:FindFirstChild("Box")
                        or curChar:FindFirstChildWhichIsA("Tool")
                    if box and curHum then
                        curHum:EquipTool(box)
                        task.wait(1.0)
                    end
                    prompt:InputHoldBegin()
                    task.wait(prompt.HoldDuration + 0.2)
                    prompt:InputHoldEnd()
                    task.wait(2.5)
                end
            else
                for _, p in ipairs(LocationFolder:GetChildren()) do
                    local b = p:FindFirstChild("Block")
                    local pr = b and b:FindFirstChild("ProximityPrompt")
                    if pr and pr.Enabled then
                        print("Fallback: taruh paket #" .. p.Name)
                        local box = LocalPlayer.Backpack:FindFirstChild("Box")
                            or curChar:FindFirstChild("Box")
                            or curChar:FindFirstChildWhichIsA("Tool")
                        if box and curHum then
                            curHum:EquipTool(box)
                            task.wait(1.0)
                        end
                        pr:InputHoldBegin()
                        task.wait(pr.HoldDuration + 0.2)
                        pr:InputHoldEnd()
                        task.wait(2.5)
                    end
                end
            end
        end)

        print("Paket #" .. tostring(targetNum) .. " selesai! Cek paket berikutnya...")
        task.wait(2.0)
    end

    if ServiceEventConn then
        ServiceEventConn:Disconnect()
        ServiceEventConn = nil
    end
    print("Auto Courier dihentikan.")
end
-- =============================================
-- GUI - TAB RACE
-- =============================================

local RaceTab = Window:AddTab({ Name = "Race", Icon = "gamepad" })

local SpeedSection = RaceTab:AddSection("Speed Hack")

SpeedSection:AddParagraph({
    Title = "Penting",
    Content = isMobileDevice
        and "MOBILE: Naik motor dulu â†’ Toggle ON â†’ Tekan gas â†’ boost! Lepas gas â†’ rem halus!"
        or  "PC: Naik kendaraan dulu â†’ Toggle ON â†’ Pencet W â†’ boost! Lepas W â†’ rem halus!"
})

SpeedSection:AddInput({
    Title = "Speed Value",
    Default = "100",
    Placeholder = "Enter speed (10-1000)",
    Callback = function(value)
        local speed = tonumber(value)
        if speed and speed >= 10 and speed <= 1000 then
            VehicleSpeed.SetSpeed(speed)
        end
    end
})

SpeedSection:AddToggle({
    Title = "Enable Speed Hack",
    Default = false,
    Callback = function(on)
        if on then VehicleSpeed.Start() else VehicleSpeed.Stop() end
    end
})

local SlowSection = RaceTab:AddSection("Slow Race (Gradual)")

SlowSection:AddParagraph({
    Title = "Info",
    Content = isMobileDevice
        and "MOBILE: Naik motor dulu â†’ Toggle ON â†’ Tekan gas â†’ speed naik pelan! Lepas gas â†’ rem halus!"
        or  "PC: Speed naik pelan-pelan sampai Max Speed. Lepas W â†’ rem halus! Makin besar Accel, makin cepat naiknya!"
})

SlowSection:AddInput({
    Title = "Max Speed",
    Default = "200",
    Placeholder = "Batas max speed (10-1000)",
    Callback = function(value)
        local v = tonumber(value)
        if v and v >= 10 and v <= 1000 then
            SlowRace.MaxSpeed = v
        end
    end
})

SlowSection:AddInput({
    Title = "Acceleration (kelipatan)",
    Default = "3",
    Placeholder = "Kelipatan akselerasi (1-20)",
    Callback = function(value)
        local v = tonumber(value)
        if v and v >= 1 and v <= 20 then
            SlowRace.AccelMultiplier = v
        end
    end
})

SlowSection:AddToggle({
    Title = "Enable Slow Race",
    Default = false,
    Callback = function(on)
        if on then SlowRace.Start() else SlowRace.Stop() end
    end
})

-- =============================================
-- GUI - TAB GARAGE
-- =============================================

local initialCarList = getCarList()

local GarageTab = Window:AddTab({ Name = "Garage", Icon = "menu" })
local GarageSection = GarageTab:AddSection("Spawn Kendaraan")

local carDropdown = GarageSection:AddDropdown({
    Title = "Pilih Kendaraan",
    Content = "Pilih kendaraan yang mau di-spawn",
    Multi = false,
    Options = initialCarList,
    Default = initialCarList[1],
    Callback = function(selected)
        if selected ~= "Refresh dulu..." then
            SpawnCar.SelectedCar = selected
        end
    end
})
SpawnCar.SelectedCar = initialCarList[1] ~= "Refresh dulu..." and initialCarList[1] or nil

GarageSection:AddButton({
    Title = "Refresh List Kendaraan",
    Callback = function()
        local cars = getCarList()
        if cars[1] ~= "Refresh dulu..." then
            pcall(function()
                carDropdown:Refresh(cars, cars[1])
            end)
            SpawnCar.SelectedCar = cars[1]
        end
    end
})

GarageSection:AddButton({
    Title = "Spawn Kendaraan",
    Callback = function() SpawnCar.Spawn() end
})

GarageSection:AddButton({
    Title = "Despawn Kendaraan",
    Callback = function() SpawnCar.Despawn() end
})

GarageSection:AddButton({
    Title = "Ride Motor",
    Callback = function() rideMotor() end
})

GarageSection:AddToggle({
    Title = "Auto Ride after Spawn",
    Default = false,
    Callback = function(on)
        SpawnCar.AutoRide = on
    end
})

GarageSection:AddToggle({
    Title = "Auto Ride Always",
    Default = false,
    Callback = function(on)
        SpawnCar.AutoRideAlways = on
        if on then SpawnCar.StartAutoRideAlways()
        else SpawnCar.StopAutoRideAlways() end
    end
})

-- =============================================
-- AUTO MONEY LIMIT + REJOIN MODULE
-- =============================================

local AutoMoneyLimit = (function()
    local AML = {}
    AML.Enabled = false
    AML.LimitAmount = 70000000 -- default 70 juta
    AML.Thread = nil

    -- Parse "Rp 70.000.000" atau "Rp70000000" â†’ angka
    local function parseRupiah(text)
        if not text or text == "" then return 0 end
        -- Hapus semua non-digit (Rp, titik, koma, spasi, dll)
        local clean = text:gsub("%D", "")
        return tonumber(clean) or 0
    end

    function AML.Start()
        if AML.Enabled then return end
        AML.Enabled = true

        AML.Thread = task.spawn(function()
            print("[AutoMoneyLimit] Monitor uang aktif. Limit:", AML.LimitAmount)
            while AML.Enabled do
                task.wait(1)
                if not AML.Enabled then break end

                local ok, amount = pcall(function()
                    local label = LocalPlayer.PlayerGui
                        :WaitForChild("BaristaMissionUI", 1)
                        :WaitForChild("Container", 1)
                        :WaitForChild("MainFrame", 1)
                        :WaitForChild("Frame", 1)
                        :WaitForChild("TotalLabel", 1)
                    return parseRupiah(label.Text)
                end)

                if ok and amount >= AML.LimitAmount then
                    print("[AutoMoneyLimit] Uang " .. tostring(amount) .. " >= limit! Kick dan rejoin...")
                    AML.Enabled = false
                    task.wait(0.5)
                    -- Kick sendiri â†’ trigger Auto Reconnect + Auto Execute
                    LocalPlayer:Kick("[AutoLimit] Uang sudah mencapai batas, rejoin otomatis!")
                    return
                end
            end
            print("[AutoMoneyLimit] Monitor berhenti")
        end)
    end

    function AML.Stop()
        AML.Enabled = false
        if AML.Thread then
            task.cancel(AML.Thread)
            AML.Thread = nil
        end
    end

    function AML.SetLimit(amount)
        AML.LimitAmount = amount
    end

    return AML
end)()

-- =============================================
-- GUI - TAB JOB
-- =============================================

local JobTab = Window:AddTab({ Name = "Job", Icon = "payment" })
local JobSection = JobTab:AddSection("Auto Job Barista")

JobSection:AddButton({
    Title = "Accept Job Barista",
    Callback = function()
        setJob(BaristaJob)
    end
})

JobSection:AddToggle({
    Title = "Auto Work Barista",
    Default = false,
    Callback = function(on)
        if on then
            task.spawn(startBaristaLoop)
        else
            baristaRunning = false
        end
    end
})

JobSection:AddParagraph({
    Title = "Auto Rejoin by Money Limit",
    Content = "Otomatis kick + rejoin kalau TotalLabel sudah capai limit.\nPastikan Auto Reconnect & Auto Execute sudah ON sebelum aktifin fitur ini!"
})

JobSection:AddInput({
    Title = "Money Limit (angka)",
    Default = "70000000",
    Placeholder = "Contoh: 70000000",
    Callback = function(value)
        local num = tonumber(value)
        if num and num > 0 then
            AutoMoneyLimit.SetLimit(num)
            print("[AutoMoneyLimit] Limit diset ke:", num)
        end
    end
})

JobSection:AddToggle({
    Title = "Auto Rejoin by Money Limit",
    Default = false,
    Callback = function(on)
        if on then
            AutoMoneyLimit.Start()
        else
            AutoMoneyLimit.Stop()
        end
    end
})

local CourierSection = JobTab:AddSection("Auto Job Courier")

CourierSection:AddButton({
    Title = "Accept Job Courier",
    Callback = function()
        setJob(CourierJob)
    end
})

CourierSection:AddToggle({
    Title = "Auto Work Courier",
    Default = false,
    Callback = function(on)
        if on then
            task.spawn(startCourierLoop)
        else
            courierRunning = false
        end
    end
})


-- =============================================
-- CHARACTER ADDED
-- =============================================

local function onCharacterAdded(char)
    Character = char
end

LocalPlayer.CharacterAdded:Connect(onCharacterAdded)
if Character then onCharacterAdded(Character) end

-- =============================================
-- HIDE STATS MODULE
-- =============================================

local HideStats = (function()
    local HS = {}

    local enabled = false
    local FakeName = "King Vypers"
    local FakeRank = "King Vypers ðŸ‘‘"
    local updateLoop = nil

    local function getRankTags()
        local char = LocalPlayer.Character
        if not char then return nil end
        local head = char:FindFirstChild("Head")
        if not head then return nil end
        return head:FindFirstChild("RankTags")
    end

    local function updateStats()
        if not enabled then return end
        local rankTags = getRankTags()
        if not rankTags then return end

        local username = rankTags:FindFirstChild("Player_Username")
        local rank = rankTags:FindFirstChild("Player_Rank")

        if username then
            username.Text = FakeName
            local shadow = username:FindFirstChild("Shadow")
            if shadow then shadow.Text = FakeName end
        end
        if rank then rank.Text = FakeRank end
    end

    local function restoreStats()
        local rankTags = getRankTags()
        if not rankTags then return end

        local username = rankTags:FindFirstChild("Player_Username")
        local rank = rankTags:FindFirstChild("Player_Rank")

        if username then
            username.Text = LocalPlayer.Name
            local shadow = username:FindFirstChild("Shadow")
            if shadow then shadow.Text = LocalPlayer.Name end
        end
        if rank then rank.Text = "Member" end
    end

    local function startLoop()
        if updateLoop then return end
        updateLoop = true
        task.spawn(function()
            while updateLoop do
                task.wait(0.2)
                if enabled then updateStats() end
            end
        end)
    end

    function HS.Enable()
        enabled = true
        startLoop()
        updateStats()
    end

    function HS.Disable()
        enabled = false
        updateLoop = false
        restoreStats()
    end

    function HS.SetFakeName(name)
        FakeName = name or LocalPlayer.Name
        if enabled then updateStats() end
    end

    function HS.SetFakeRank(rank)
        FakeRank = rank or "Member"
        if enabled then updateStats() end
    end

    LocalPlayer.CharacterAdded:Connect(function()
        task.wait(1)
        if enabled then updateStats() end
    end)

    return HS
end)()

-- =============================================
-- ANTI-AFK MODULE
-- =============================================

local AntiAFK = (function()
    local AA = {
        Enabled = false,
        Thread = nil,
    }

    function AA.Start()
        if AA.Enabled then return end
        AA.Enabled = true

        AA.Thread = task.spawn(function()
            while AA.Enabled do
                task.wait(240)
                if not AA.Enabled then break end
                pcall(function()
                    local cam = Workspace.CurrentCamera
                    local orig = cam.CFrame
                    cam.CFrame = orig * CFrame.Angles(0, 0.01, 0)
                    task.wait(0.1)
                    cam.CFrame = orig
                end)
            end
        end)
    end

    function AA.Stop()
        if not AA.Enabled then return end
        AA.Enabled = false
        if AA.Thread then
            task.cancel(AA.Thread)
            AA.Thread = nil
        end
    end

    return AA
end)()

-- =============================================
-- ANTI-STAFF MODULE
-- =============================================

local AntiStaff = (function()
    local AS = {}
    AS.Active = false

    local GROUP_ID = 35102746
    local STAFF_RANKS = {
        [2]=true, [3]=true, [4]=true, [75]=true, [79]=true,
        [145]=true, [250]=true, [252]=true, [254]=true, [255]=true,
        [55]=true, [30]=true, [35]=true, [100]=true, [76]=true
    }

    local function checkLoop()
        while AS.Active do
            for _, player in ipairs(Players:GetPlayers()) do
                if player ~= LocalPlayer then
                    local ok, rank = pcall(function()
                        return player:GetRankInGroup(GROUP_ID)
                    end)
                    if ok and STAFF_RANKS[rank] then
                        LocalPlayer:Kick("Staff Detected! Auto Kicked for Safety.")
                        return
                    end
                end
            end
            task.wait(1)
        end
    end

    function AS.Start()
        if AS.Active then return end
        AS.Active = true
        task.spawn(checkLoop)
    end

    function AS.Stop()
        AS.Active = false
    end

    return AS
end)()

-- =============================================
-- AUTO RECONNECT + AUTO EXECUTE MODULE
-- =============================================

local AutoRejoin = (function()
    local AR = {}
    AR.Enabled = false
    AR.AutoExecEnabled = false

    local TeleportService = game:GetService("TeleportService")
    local Players = game:GetService("Players")
    local LocalPlayer = Players.LocalPlayer
    local disconnectSetup = false
    local hasTriggered = false

    -- URL script yang akan di-execute otomatis setelah rejoin
    -- WARNING: Ganti SCRIPT_URL dengan URL script UTAMA (misal pastebin/github raw cobadds.lua lo)
    -- JANGAN pakai URL vyperui.lua karena itu cuma UI-nya saja!
    local SCRIPT_URL = "https://gitlab.com/blazars1/KingVYpersPrem/-/raw/main/KingVypersPrem.lua" 
    local EXEC_DELAY = 20 -- detik tunggu sebelum execute setelah rejoin (dilebihin dikit biar game load)

    -- Cari queue_on_teleport dari berbagai executor secara aman
    local function getQueueOnTeleport()
        local getQueue = nil
        pcall(function() getQueue = queue_on_teleport end)
        if getQueue then return getQueue end
        pcall(function() getQueue = queueonteleport end)
        if getQueue then return getQueue end
        pcall(function() getQueue = syn and syn.queue_on_teleport end)
        if getQueue then return getQueue end
        pcall(function() getQueue = fluxus and fluxus.queue_on_teleport end)
        if getQueue then return getQueue end
        return nil
    end

    local function setupAutoExecuteQueue()
        local queueTeleport = getQueueOnTeleport()
        if not queueTeleport then return false end
        if not AR.AutoExecEnabled then return false end

        local autoExecCode = string.format([[
            task.wait(%d)
            pcall(function()
                loadstring(game:HttpGet("%s"))()
            end)
        ]], EXEC_DELAY, SCRIPT_URL)

        local ok = pcall(function() queueTeleport(autoExecCode) end)
        return ok
    end

    local function doRejoin()
        if hasTriggered then return end
        if not AR.Enabled then return end
        hasTriggered = true

        -- Queue auto execute dulu sebelum teleport kalau fitur aktif
        if AR.AutoExecEnabled then
            setupAutoExecuteQueue()
        end

        task.spawn(function()
            while true do
                pcall(function()
                    TeleportService:Teleport(game.PlaceId, LocalPlayer)
                end)
                task.wait(3) -- Loop terus misal teleport gagal
            end
        end)
    end

    local function setupDetection()
        if disconnectSetup then return end
        disconnectSetup = true

        -- Method 1: GuiService ErrorMessageChanged
        pcall(function()
            game:GetService("GuiService").ErrorMessageChanged:Connect(function(message)
                if message and message ~= "" and AR.Enabled then
                    task.wait(1)
                    doRejoin()
                end
            end)
        end)

        -- Method 2: CoreGui RobloxPromptGui â€” popup "You were kicked" / error
        pcall(function()
            local CoreGui = game:GetService("CoreGui")
            local RobloxPromptGui = CoreGui:WaitForChild("RobloxPromptGui", 5)
            if RobloxPromptGui then
                local promptOverlay = RobloxPromptGui:WaitForChild("promptOverlay", 5)
                if promptOverlay then
                    promptOverlay.ChildAdded:Connect(function(child)
                        if child.Name == "ErrorPrompt" and AR.Enabled then
                            task.wait(0.5)
                            doRejoin()
                        end
                    end)
                end
            end
        end)

        -- Method 3: LocalPlayer.Idled â€” kicked karena idle terlalu lama
        pcall(function()
            LocalPlayer.Idled:Connect(function(t)
                if t > 1150 and AR.Enabled then -- 1150 detik biar gakeduluan Roblox
                    doRejoin()
                end
            end)
        end)

        -- Method 4: OnTeleport â€” fallback
        pcall(function()
            LocalPlayer.OnTeleport:Connect(function(state)
                if state == Enum.TeleportState.RequestedFromServer and AR.Enabled then
                    task.wait(1)
                    doRejoin()
                end
            end)
        end)
    end

    function AR.Start()
        if AR.Enabled then return end
        AR.Enabled = true
        hasTriggered = false
        setupDetection()
        
        if AR.AutoExecEnabled then
            setupAutoExecuteQueue()
        end
    end

    function AR.Stop()
        AR.Enabled = false
        hasTriggered = false
    end

    function AR.EnableAutoExec()
        AR.AutoExecEnabled = true
        if AR.Enabled then
            setupAutoExecuteQueue()
        end
    end

    function AR.DisableAutoExec()
        AR.AutoExecEnabled = false
    end

    function AR.SetScriptURL(url)
        if type(url) == "string" and url ~= "" then
            SCRIPT_URL = url
        end
    end

    function AR.IsQueueSupported()
        return getQueueOnTeleport() ~= nil
    end

    return AR
end)()

-- =============================================
-- POTATO MODE MODULE
-- =============================================

local PotatoMode = (function()
    local PM = {}
    PM.Enabled = false

    local Lighting = game:GetService("Lighting")
    local StarterGui = game:GetService("StarterGui")
    local Terrain = Workspace:FindFirstChildOfClass("Terrain")

    local originalStates = { lighting = {}, waterProperties = {}, camera = {} }
    local pmConnections = {}
    local processedObjects = setmetatable({}, {__mode = "k"})

    local DESTROY_CLASSES = {
        BloomEffect=true, BlurEffect=true, ColorCorrectionEffect=true,
        SunRaysEffect=true, DepthOfFieldEffect=true, Atmosphere=true,
    }

    local function shouldDestroy(obj) return DESTROY_CLASSES[obj.ClassName] end

    local function isInVehicle(obj)
        local parent = obj.Parent
        while parent and parent ~= Workspace do
            if parent.Name:find("Montors") or parent.Name:find("Vehicle") or parent.Name:find("Car") then
                return true
            end
            parent = parent.Parent
        end
        return false
    end

    local function safeDestroy(obj)
        local ok, locked = pcall(function() return obj.Locked end)
        if ok and locked then return end
        if isInVehicle(obj) then return end
        pcall(function() obj:Destroy() end)
    end

    local function optimizeObject(obj)
        if not PM.Enabled or processedObjects[obj] then return end
        processedObjects[obj] = true
        if shouldDestroy(obj) then safeDestroy(obj) return end
        pcall(function()
            if obj:IsA("BasePart") then
                obj.Material = Enum.Material.SmoothPlastic
                obj.CastShadow = false
                obj.Reflectance = 0
                obj.TopSurface = Enum.SurfaceType.SmoothNoOutlines
                obj.BottomSurface = Enum.SurfaceType.SmoothNoOutlines
                obj.LeftSurface = Enum.SurfaceType.SmoothNoOutlines
                obj.RightSurface = Enum.SurfaceType.SmoothNoOutlines
                obj.FrontSurface = Enum.SurfaceType.SmoothNoOutlines
                obj.BackSurface = Enum.SurfaceType.SmoothNoOutlines
            elseif obj:IsA("Sound") then
                obj.Volume = 0
            end
        end)
    end

    local function optimizeCharacter(character)
        if not character or processedObjects[character] then return end
        processedObjects[character] = true
        pcall(function()
            for _, obj in ipairs(character:GetDescendants()) do
                if shouldDestroy(obj) then
                    local okL, isLocked = pcall(function() return obj.Locked end)
                    if not (okL and isLocked) then
                        pcall(function() obj:Destroy() end)
                    end
                elseif obj:IsA("BasePart") then
                    if obj.Name == "Head" then obj.Transparency = 1 end
                    obj.Material = Enum.Material.SmoothPlastic
                    obj.CastShadow = false
                    obj.CanCollide = obj.Name == "HumanoidRootPart" or obj.Name == "Head"
                    obj.Reflectance = 0
                    obj.TopSurface = Enum.SurfaceType.SmoothNoOutlines
                    obj.BottomSurface = Enum.SurfaceType.SmoothNoOutlines
                    obj.LeftSurface = Enum.SurfaceType.SmoothNoOutlines
                    obj.RightSurface = Enum.SurfaceType.SmoothNoOutlines
                    obj.FrontSurface = Enum.SurfaceType.SmoothNoOutlines
                    obj.BackSurface = Enum.SurfaceType.SmoothNoOutlines
                elseif obj:IsA("Humanoid") then
                    for _, t in ipairs(obj:GetPlayingAnimationTracks()) do t:Stop() end
                    obj.HealthDisplayDistance = 0
                    obj.NameDisplayDistance = 0
                elseif obj:IsA("Sound") then
                    obj.Volume = 0
                end
            end
        end)
    end

    function PM.Enable()
        if PM.Enabled then return end
        PM.Enabled = true

        task.spawn(function()
            local all = Workspace:GetDescendants()
            for i = 1, #all, 200 do
                if not PM.Enabled then break end
                for j = i, math.min(i+199, #all) do optimizeObject(all[j]) end
                task.wait()
            end
        end)

        if LocalPlayer.Character then optimizeCharacter(LocalPlayer.Character) end

        table.insert(pmConnections, LocalPlayer.CharacterAdded:Connect(function(char)
            if PM.Enabled then task.wait(0.2) optimizeCharacter(char) end
        end))

        if Terrain then
            pcall(function()
                originalStates.waterProperties = {
                    WaterReflectance = Terrain.WaterReflectance,
                    WaterWaveSize = Terrain.WaterWaveSize,
                    WaterWaveSpeed = Terrain.WaterWaveSpeed,
                    WaterTransparency = Terrain.WaterTransparency
                }
                Terrain.WaterWaveSize = 0
                Terrain.WaterWaveSpeed = 0
                Terrain.WaterReflectance = 0
                Terrain.WaterTransparency = 1
                Terrain.Decoration = false
            end)
            local clouds = Terrain:FindFirstChildOfClass("Clouds")
            if clouds then clouds:Destroy() end
        end

        for _, sky in ipairs(Lighting:GetChildren()) do
            if sky:IsA("Sky") then
                sky.SkyboxBk="" sky.SkyboxDn="" sky.SkyboxFt=""
                sky.SkyboxLf="" sky.SkyboxRt="" sky.SkyboxUp=""
                sky.StarCount=0 sky.SunAngularSize=0 sky.MoonAngularSize=0
                sky.CelestialBodiesShown=false
            end
        end

        local atmosphere = Lighting:FindFirstChildOfClass("Atmosphere")
        if atmosphere then atmosphere:Destroy() end

        originalStates.lighting = {
            GlobalShadows = Lighting.GlobalShadows,
            Brightness = Lighting.Brightness,
            Technology = Lighting.Technology
        }
        Lighting.GlobalShadows = false
        Lighting.FogEnd = 9e9
        Lighting.FogStart = 0
        Lighting.Brightness = 0
        Lighting.OutdoorAmbient = Color3.new(1,1,1)
        Lighting.Ambient = Color3.new(1,1,1)
        Lighting.Technology = Enum.Technology.Legacy
        Lighting.EnvironmentDiffuseScale = 0
        Lighting.EnvironmentSpecularScale = 0
        Lighting.ShadowSoftness = 0

        for _, effect in ipairs(Lighting:GetChildren()) do
            if effect:IsA("PostEffect") then effect.Enabled = false end
        end

        pcall(function() settings().Rendering.QualityLevel = Enum.QualityLevel.Level01 end)
        pcall(function()
            local cam = Workspace.CurrentCamera
            originalStates.camera = { FieldOfView = cam.FieldOfView }
            cam.FieldOfView = 70
        end)

        table.insert(pmConnections, Workspace.DescendantAdded:Connect(function(obj)
            if PM.Enabled then
                if shouldDestroy(obj) then safeDestroy(obj)
                else task.defer(optimizeObject, obj) end
            end
        end))

        pcall(function()
            StarterGui:SetCoreGuiEnabled(Enum.CoreGuiType.PlayerList, false)
            StarterGui:SetCoreGuiEnabled(Enum.CoreGuiType.EmotesMenu, false)
        end)
    end

    function PM.Disable()
        if not PM.Enabled then return end
        PM.Enabled = false

        if Terrain and originalStates.waterProperties then
            pcall(function()
                Terrain.WaterReflectance = originalStates.waterProperties.WaterReflectance or 0
                Terrain.WaterWaveSize = originalStates.waterProperties.WaterWaveSize or 0
                Terrain.WaterWaveSpeed = originalStates.waterProperties.WaterWaveSpeed or 0
                Terrain.WaterTransparency = originalStates.waterProperties.WaterTransparency or 0
                Terrain.Decoration = true
            end)
        end

        if originalStates.lighting.GlobalShadows ~= nil then
            Lighting.GlobalShadows = originalStates.lighting.GlobalShadows
            Lighting.Brightness = originalStates.lighting.Brightness
            Lighting.Technology = originalStates.lighting.Technology
        end

        if originalStates.camera.FieldOfView then
            pcall(function() Workspace.CurrentCamera.FieldOfView = originalStates.camera.FieldOfView end)
        end

        pcall(function() settings().Rendering.QualityLevel = Enum.QualityLevel.Automatic end)
        pcall(function()
            StarterGui:SetCoreGuiEnabled(Enum.CoreGuiType.PlayerList, true)
            StarterGui:SetCoreGuiEnabled(Enum.CoreGuiType.EmotesMenu, true)
        end)

        for _, conn in ipairs(pmConnections) do conn:Disconnect() end
        pmConnections = {}
        processedObjects = setmetatable({}, {__mode = "k"})
        originalStates = { lighting = {}, waterProperties = {}, camera = {} }
        pcall(function() collectgarbage("collect") end)
    end

    return PM
end)()

-- =============================================
-- DISABLE RENDERING MODULE
-- =============================================

local DisableRendering = (function()
    local DR = {}
    DR.Enabled = false
    local renderConn = nil

    function DR.Start()
        if DR.Enabled then return end
        DR.Enabled = true
        renderConn = RunService.RenderStepped:Connect(function()
            pcall(function() RunService:Set3dRenderingEnabled(false) end)
        end)
    end

    function DR.Stop()
        if not DR.Enabled then return end
        DR.Enabled = false
        if renderConn then renderConn:Disconnect() renderConn = nil end
        pcall(function() RunService:Set3dRenderingEnabled(true) end)
    end

    return DR
end)()

-- =============================================
-- UNLOCK FPS MODULE
-- =============================================

local UnlockFPS = (function()
    local UF = {}
    UF.Enabled = false
    UF.CurrentCap = 60

    function UF.SetCap(fps)
        UF.CurrentCap = fps
        if UF.Enabled and setfpscap then setfpscap(fps) end
    end

    function UF.Start()
        if UF.Enabled then return end
        UF.Enabled = true
        if setfpscap then setfpscap(UF.CurrentCap) end
    end

    function UF.Stop()
        if not UF.Enabled then return end
        UF.Enabled = false
        if setfpscap then setfpscap(60) end
    end

    return UF
end)()

-- =============================================
-- GUI - TAB SETTINGS
-- =============================================

local FreecamTab = Window:AddTab({ Name = "Settings", Icon = "settings" })

local HideStatsSection = FreecamTab:AddSection("Hide Stats")

HideStatsSection:AddToggle({
    Title = "Enable Hide Stats",
    Default = false,
    Callback = function(on)
        if on then HideStats.Enable() else HideStats.Disable() end
    end
})

HideStatsSection:AddInput({
    Title = "Fake Name",
    Default = "King Vypers",
    Placeholder = "Nama palsu",
    Callback = function(value)
        HideStats.SetFakeName(value)
    end
})

HideStatsSection:AddInput({
    Title = "Fake Rank",
    Default = "King Vypers ðŸ‘‘",
    Placeholder = "Rank palsu",
    Callback = function(value)
        HideStats.SetFakeRank(value)
    end
})

-- =============================================
-- GUI - PERFORMANCE SECTION (di Settings tab)
-- =============================================

local PerformanceSection = FreecamTab:AddSection("Performance")

PerformanceSection:AddToggle({
    Title = "FPS Booster (Potato Mode)",
    Default = false,
    Callback = function(on)
        if on then PotatoMode.Enable() else PotatoMode.Disable() end
    end
})

PerformanceSection:AddToggle({
    Title = "Disable 3D Rendering",
    Default = false,
    Callback = function(on)
        if on then DisableRendering.Start() else DisableRendering.Stop() end
    end
})

local selectedFpsCap = 60

PerformanceSection:AddDropdown({
    Title = "FPS Cap",
    Options = {"60", "90", "120", "240"},
    Default = "60",
    Callback = function(value)
        selectedFpsCap = tonumber(value) or 60
        UnlockFPS.SetCap(selectedFpsCap)
    end
})

PerformanceSection:AddToggle({
    Title = "Enable FPS Unlock",
    Default = false,
    Callback = function(on)
        if on then
            UnlockFPS.CurrentCap = selectedFpsCap
            UnlockFPS.Start()
        else
            UnlockFPS.Stop()
        end
    end
})

-- =============================================
-- GUI - PROTECTION SECTION (di Settings tab)
-- =============================================

local ProtectionSection = FreecamTab:AddSection("Protection")

ProtectionSection:AddToggle({
    Title = "Anti-AFK",
    Default = false,
    Callback = function(on)
        if on then AntiAFK.Start() else AntiAFK.Stop() end
    end
})

ProtectionSection:AddToggle({
    Title = "Anti Staff (Auto Kick)",
    Default = false,
    Callback = function(on)
        if on then AntiStaff.Start() else AntiStaff.Stop() end
    end
})

-- =============================================
-- GUI - AUTO RECONNECT + AUTO EXECUTE SECTION
-- =============================================

local ReconnectSection = FreecamTab:AddSection("Auto Reconnect & Execute")

ReconnectSection:AddParagraph({
    Title = "Info",
    Content = "Auto Reconnect: rejoin otomatis kena kick/disconnect.\nAuto Execute: jalanin script ini lagi otomatis setelah rejoin (butuh executor yang support queue_on_teleport)."
})

-- Toggle combined: Auto Reconnect
ReconnectSection:AddToggle({
    Title = "Enable Auto Reconnect",
    Default = false,
    Callback = function(on)
        if on then
            AutoRejoin.Start()
        else
            AutoRejoin.Stop()
        end
    end
})

-- Toggle: Auto Execute setelah rejoin
-- Kalau ON, script King Vypers akan otomatis ke-load lagi setelah rejoin
ReconnectSection:AddToggle({
    Title = "Auto Execute Setelah Rejoin",
    Default = false,
    Callback = function(on)
        if on then
            if AutoRejoin.IsQueueSupported() then
                AutoRejoin.EnableAutoExec()
            else
                -- Executor tidak support queue_on_teleport, fitur tidak akan jalan
                -- Toggle tetap bisa di-ON tapi tidak akan ada efek
                AutoRejoin.EnableAutoExec()
            end
        else
            AutoRejoin.DisableAutoExec()
        end
    end
})

ReconnectSection:AddParagraph({
    Title = "Auto Execute URL",
    Content = "Script URL sudah di-set ke URL King Vypers default.\nEdit variable SCRIPT_URL di module AutoRejoin kalau mau ganti."
})

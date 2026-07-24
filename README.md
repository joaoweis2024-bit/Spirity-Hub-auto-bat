local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UIS = game:GetService("UserInputService")
local TweenService = game:GetService("TweenService")
local HttpService = game:GetService("HttpService")
local LP = Players.LocalPlayer

-- // THEME (BLACK & WHITE / SPIRIT DARK MODE)
local MainBg       = Color3.fromRGB(15, 15, 15) -- Fundo principal preto
local RowBg        = Color3.fromRGB(25, 25, 25) -- Fundo da linha cinza muito escuro
local Accent       = Color3.fromRGB(255, 255, 255) -- Branco para os botões ativados
local AccentText   = Color3.fromRGB(0, 0, 0) -- Texto preto para contrastar no fundo branco
local White        = Color3.fromRGB(255, 255, 255) -- Texto branco padrão
local DimText      = Color3.fromRGB(150, 150, 150) -- Texto cinza para subtítulos
local OffBg        = Color3.fromRGB(40, 40, 40) -- Cinza escuro para estado "Desativado"

-- // STATE
local State = {
    autoBatToggled = false,
    hittingCooldown = false,
}

local Keys = {
    autoBat = Enum.KeyCode.E,
    autoBatType = "Keyboard",
}

local h, hrp = nil, nil

-- // CONFIG
local function saveConfig()
    local cfg = {
        autoBatKey = Keys.autoBat.Name,
        autoBatKeyType = Keys.autoBatType,
    }
    pcall(function() 
        writefile("SpiritHubBatConfig.json", HttpService:JSONEncode(cfg)) 
    end)
end

local function loadConfig()
    local hasFile = false
    pcall(function() hasFile = isfile("SpiritHubBatConfig.json") end)
    if not hasFile then return end
    
    local ok, cfg = pcall(function() 
        return HttpService:JSONDecode(readfile("SpiritHubBatConfig.json")) 
    end)
    if not ok or not cfg then return end
    
    if cfg.autoBatKey and Enum.KeyCode[cfg.autoBatKey] then
        Keys.autoBat = Enum.KeyCode[cfg.autoBatKey]
    end
    
    if cfg.autoBatKeyType then
        Keys.autoBatType = cfg.autoBatKeyType
    end
end

loadConfig()

-- // CLEANUP OLD GUI
for _, name in pairs({"EnvyAutoBatDesyncGUI", "MwvaneNewaBatDesyncGUI", "PhazeAutoBatDesyncGUI", "VisionBatGui", "SpiritHubBatGui"}) do
    local old = game:GetService("CoreGui"):FindFirstChild(name)
    if old then old:Destroy() end
end

-- // GUI
local gui = Instance.new("ScreenGui")
gui.Name = "SpiritHubBatGui"
gui.ResetOnSpawn = false
gui.IgnoreGuiInset = true
gui.Parent = LP:WaitForChild("PlayerGui")

local main = Instance.new("Frame", gui)
main.Size = UDim2.new(0, 220, 0, 145)
main.Position = UDim2.new(0.5, -110, 0.5, -72)
main.BackgroundColor3 = MainBg
main.BorderSizePixel = 0
main.Active = true
Instance.new("UICorner", main).CornerRadius = UDim.new(0, 12)

-- // DRAG SYSTEM
do
    local dragging, dragInput, dragStart, mainStart = false, nil, nil, nil
    main.InputBegan:Connect(function(inp)
        if inp.UserInputType == Enum.UserInputType.MouseButton1 or inp.UserInputType == Enum.UserInputType.Touch then
            dragging = true
            dragStart = inp.Position
            mainStart = main.Position
            inp.Changed:Connect(function()
                if inp.UserInputState == Enum.UserInputState.End then dragging = false end
            end)
        end
    end)
    main.InputChanged:Connect(function(inp)
        if inp.UserInputType == Enum.UserInputType.MouseMovement or inp.UserInputType == Enum.UserInputType.Touch then
            dragInput = inp
        end
    end)
    UIS.InputChanged:Connect(function(inp)
        if inp == dragInput and dragging then
            local dx = inp.Position.X - dragStart.X
            local dy = inp.Position.Y - dragStart.Y
            main.Position = UDim2.new(mainStart.X.Scale, mainStart.X.Offset + dx, mainStart.Y.Scale, mainStart.Y.Offset + dy)
        end
    end)
end

-- // UI ELEMENTS

-- Title
local titleLbl = Instance.new("TextLabel", main)
titleLbl.Size = UDim2.new(1, -16, 0, 22)
titleLbl.Position = UDim2.new(0, 8, 0, 12)
titleLbl.BackgroundTransparency = 1
titleLbl.Text = "Spirit Hub Anti Bat Bypass"
titleLbl.TextColor3 = White
titleLbl.Font = Enum.Font.GothamBold
titleLbl.TextSize = 12
titleLbl.TextXAlignment = Enum.TextXAlignment.Left

-- Subtitle
local subLbl = Instance.new("TextLabel", main)
subLbl.Size = UDim2.new(1, -16, 0, 16)
subLbl.Position = UDim2.new(0, 8, 0, 34)
subLbl.BackgroundTransparency = 1
subLbl.Text = "discord.gg/spirithub"
subLbl.TextColor3 = DimText
subLbl.Font = Enum.Font.Gotham
subLbl.TextSize = 9
subLbl.TextXAlignment = Enum.TextXAlignment.Left

-- Keybind Row Background
local keyRow = Instance.new("Frame", main)
keyRow.Size = UDim2.new(1, -16, 0, 32)
keyRow.Position = UDim2.new(0, 8, 0, 58)
keyRow.BackgroundColor3 = RowBg
keyRow.BorderSizePixel = 0
Instance.new("UICorner", keyRow).CornerRadius = UDim.new(0, 6)

-- Keybind Label
local keyLbl = Instance.new("TextLabel", keyRow)
keyLbl.Size = UDim2.new(0, 60, 1, 0)
keyLbl.Position = UDim2.new(0, 10, 0, 0)
keyLbl.BackgroundTransparency = 1
keyLbl.Text = "Keybind"
keyLbl.TextColor3 = DimText
keyLbl.Font = Enum.Font.GothamMedium
keyLbl.TextSize = 9
keyLbl.TextXAlignment = Enum.TextXAlignment.Left

-- Keybind Button
local keyBtn = Instance.new("TextButton", keyRow)
keyBtn.Size = UDim2.new(0, 44, 0, 20)
keyBtn.Position = UDim2.new(1, -54, 0.5, -10)
keyBtn.BackgroundColor3 = Accent
keyBtn.Text = Keys.autoBat.Name
keyBtn.TextColor3 = AccentText
keyBtn.Font = Enum.Font.GothamBold
keyBtn.TextSize = 9
Instance.new("UICorner", keyBtn).CornerRadius = UDim.new(0, 6)

-- Action Button (Enable/Disable)
local actionBtn = Instance.new("TextButton", main)
actionBtn.Size = UDim2.new(1, -16, 0, 34)
actionBtn.Position = UDim2.new(0, 8, 0, 102)
actionBtn.BackgroundColor3 = Accent
actionBtn.Text = "Enable"
actionBtn.TextColor3 = AccentText
actionBtn.Font = Enum.Font.GothamBold
actionBtn.TextSize = 11
Instance.new("UICorner", actionBtn).CornerRadius = UDim.new(0, 6)

-- // KEYBIND LOGIC
local kListening = false
local kConn

local function stopListen(key)
    kListening = false
    if kConn then
        kConn:Disconnect()
        kConn = nil
    end
    if key then
        Keys.autoBat = key
        keyBtn.Text = key.Name
        saveConfig()
    end
    keyBtn.BackgroundColor3 = Accent
    keyBtn.TextColor3 = AccentText
end

keyBtn.MouseButton1Click:Connect(function()
    if kListening then
        stopListen(nil)
        return
    end
    
    kListening = true
    keyBtn.Text = "..."
    keyBtn.BackgroundColor3 = MainBg
    keyBtn.TextColor3 = White
    
    kConn = UIS.InputBegan:Connect(function(inp)
        if not kListening then return end
        
        if inp.UserInputType == Enum.UserInputType.Keyboard then
            if inp.KeyCode == Enum.KeyCode.Escape then
                stopListen(nil)
                return
            end
            Keys.autoBatType = "Keyboard"
            stopListen(inp.KeyCode)
        elseif inp.UserInputType == Enum.UserInputType.Gamepad1 then
            Keys.autoBatType = "Gamepad"
            stopListen(inp.KeyCode)
        end
    end)
end)

-- // ACTION BUTTON LOGIC
local function updateActionBtn(state)
    if state then
        actionBtn.Text = "Disable"
        actionBtn.BackgroundColor3 = OffBg
        actionBtn.TextColor3 = DimText
    else
        actionBtn.Text = "Enable"
        actionBtn.BackgroundColor3 = Accent
        actionBtn.TextColor3 = AccentText
    end
end

actionBtn.MouseButton1Click:Connect(function()
    State.autoBatToggled = not State.autoBatToggled
    updateActionBtn(State.autoBatToggled)
end)

actionBtn.MouseEnter:Connect(function()
    TweenService:Create(actionBtn, TweenInfo.new(0.1), {
        BackgroundColor3 = State.autoBatToggled and Color3.fromRGB(55, 55, 55) or Color3.fromRGB(220, 220, 220)
    }):Play()
end)

actionBtn.MouseLeave:Connect(function()
    TweenService:Create(actionBtn, TweenInfo.new(0.1), {
        BackgroundColor3 = State.autoBatToggled and OffBg or Accent
    }):Play()
end)

-- // BAT LOGIC (PRESERVED)
local function getBat()
    local char = LP.Character
    if not char then return nil end
    local tool = char:FindFirstChild("Bat")
    if tool then return tool end
    local bp2 = LP:FindFirstChild("Backpack")
    if bp2 then
        tool = bp2:FindFirstChild("Bat")
        if tool then tool.Parent = char; return tool end
    end
    return nil
end

local function tryHitBat()
    if State.hittingCooldown then return end
    State.hittingCooldown = true
    pcall(function()
        local bat = getBat()
        if bat then
            bat:Activate()
            local ev = bat:FindFirstChildWhichIsA("RemoteEvent")
            if ev then ev:FireServer() end
        end
    end)
    task.delay(0.08, function() State.hittingCooldown = false end)
end

local function getClosestPlayer()
    if not hrp then return nil, math.huge end
    local cp, cd = nil, math.huge
    for _, p in pairs(Players:GetPlayers()) do
        if p ~= LP and p.Character then
            local tr = p.Character:FindFirstChild("HumanoidRootPart")
            if tr then
                local d = (hrp.Position - tr.Position).Magnitude
                if d < cd then cd = d; cp = p end
            end
        end
    end
    return cp, cd
end

-- // CHARACTER SETUP (PRESERVED)
local function setupChar(char)
    task.wait(0.1)
    h = char:WaitForChild("Humanoid", 5)
    hrp = char:WaitForChild("HumanoidRootPart", 5)
    if not h or not hrp then return end
end

LP.CharacterAdded:Connect(setupChar)
if LP.Character then task.spawn(function() setupChar(LP.Character) end) end

-- // BAT AIMBOT HEARTBEAT (PRESERVED)
RunService.Heartbeat:Connect(function()
    if not (State.autoBatToggled and h and hrp) then return end
    local target, dist = getClosestPlayer()
    if target and target.Character then
        local tr = target.Character:FindFirstChild("HumanoidRootPart")
        if tr then
            if sethiddenproperty then
                sethiddenproperty(hrp, "PhysicsRepRootPart", tr)
            end
            local targetPos = tr.Position + Vector3.new(0, 0.9, 0)
            if (hrp.Position - targetPos).Magnitude > 8 then
                hrp.CFrame = CFrame.new(targetPos)
            end
            local cam = workspace.CurrentCamera
            cam.CFrame = CFrame.new(cam.CFrame.Position, tr.Position)
            tryHitBat()
        end
    end
end)

-- // KEYBIND HANDLER (PRESERVED)
UIS.InputBegan:Connect(function(inp, gp)
    if gp then return end
    
    if Keys.autoBatType == "Keyboard" and inp.UserInputType == Enum.UserInputType.Keyboard then
        if inp.KeyCode == Keys.autoBat then
            State.autoBatToggled = not State.autoBatToggled
            updateActionBtn(State.autoBatToggled)
        end
    elseif Keys.autoBatType == "Gamepad" and inp.UserInputType == Enum.UserInputType.Gamepad1 then
        if inp.KeyCode == Keys.autoBat then
            State.autoBatToggled = not State.autoBatToggled
            updateActionBtn(State.autoBatToggled)
        end
    end
end)

print("[Spirit Hub Style Bat - Black & White Mode] Loaded successfully!")

local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer
local CoreGui = game:GetService("CoreGui")

local FILE = "Redline_KeyData.txt"
local DISCORD_LINK = "https://discord.gg/h2vy4FdTBb"

local RED = Color3.fromRGB(255, 0, 45)
local BLACK = Color3.fromRGB(10, 10, 13)
local WHITE = Color3.fromRGB(255, 255, 255)

-- KEY CONFIGURATION
local KEYS = {
    ["key12345"] = 4 * 60 * 60,
    ["Key12345"] = 4 * 60 * 60,
    ["ppoo4567890Q"] = 24 * 60 * 60
}

local savedData = nil

-- LOCAL SAVE SYSTEM
local function saveData(data)
    pcall(function()
        writefile(FILE, data)
    end)
end

local function loadData()
    local success, data = pcall(function()
        if isfile(FILE) then
            return readfile(FILE)
        end
    end)

    if success and data then
        return data
    end
end

-- UI CREATOR
local GUI = Instance.new("ScreenGui")
GUI.Name = "REDLINE_SYSTEM"
GUI.ResetOnSpawn = false

pcall(function()
    GUI.Parent = CoreGui
end)

if not GUI.Parent then
    GUI.Parent = LocalPlayer:WaitForChild("PlayerGui")
end

local function corner(obj, radius)
    local c = Instance.new("UICorner")
    c.CornerRadius = UDim.new(0, radius)
    c.Parent = obj
end

local function outline(obj)
    local s = Instance.new("UIStroke")
    s.Color = RED
    s.Thickness = 2
    s.Parent = obj
end

local function makeButton(parent, text, y)
    local b = Instance.new("TextButton")
    b.Size = UDim2.new(0.85, 0, 0, 38)
    b.Position = UDim2.new(0.075, 0, 0, y)
    b.BackgroundColor3 = Color3.fromRGB(30, 15, 20)
    b.Text = text
    b.TextColor3 = WHITE
    b.TextSize = 13
    b.Font = Enum.Font.GothamBold
    b.Parent = parent
    corner(b, 8)
    outline(b)
    return b
end

-- KEY LOGIN WINDOW
local Login = Instance.new("Frame")
Login.Size = UDim2.new(0, 280, 0, 285)
Login.Position = UDim2.new(0.5, -140, 0.5, -142)
Login.BackgroundColor3 = BLACK
Login.Active = true
Login.Draggable = true
Login.Parent = GUI

corner(Login, 12)
outline(Login)

local Header = Instance.new("TextLabel")
Header.Size = UDim2.new(1, 0, 0, 48)
Header.BackgroundColor3 = Color3.fromRGB(25, 5, 10)
Header.Text = "REDLINE // KEY SYSTEM"
Header.TextColor3 = RED
Header.TextSize = 16
Header.Font = Enum.Font.GothamBold
Header.Parent = Login

-- KEY INPUT (FIXED)
local KeyInput = Instance.new("TextBox")
KeyInput.Size = UDim2.new(0.85, 0, 0, 40)
KeyInput.Position = UDim2.new(0.075, 0, 0, 65)
KeyInput.BackgroundColor3 = Color3.fromRGB(25, 25, 30)
KeyInput.Text = ""
KeyInput.PlaceholderText = "ENTER YOUR KEY"
KeyInput.PlaceholderColor3 = Color3.fromRGB(150, 150, 150)
KeyInput.TextColor3 = WHITE
KeyInput.TextSize = 14
KeyInput.Font = Enum.Font.GothamBold
KeyInput.ClearTextOnFocus = false
KeyInput.Parent = Login

corner(KeyInput, 8)
outline(KeyInput)

local Remember = false

local RememberButton = makeButton(Login, "REMEMBER ME: OFF", 115)
local LoginButton = makeButton(Login, "UNLOCK REDLINE", 160)
local DiscordButton = makeButton(Login, "JOIN DISCORD", 205)

local Status = Instance.new("TextLabel")
Status.Size = UDim2.new(1, -10, 0, 25)
Status.Position = UDim2.new(0, 5, 0, 247)
Status.BackgroundTransparency = 1
Status.Text = "Enter your access key"
Status.TextColor3 = WHITE
Status.TextSize = 11
Status.Font = Enum.Font.Gotham
Status.Parent = Login

-- REMEMBER ME
RememberButton.MouseButton1Click:Connect(function()
    Remember = not Remember
    RememberButton.Text = "REMEMBER ME: " ..
        (Remember and "ON" or "OFF")
end)

-- DISCORD BUTTON
DiscordButton.MouseButton1Click:Connect(function()
    local success = pcall(function()
        assert(type(setclipboard) == "function", "Clipboard unavailable")
        setclipboard(DISCORD_LINK)
    end)

    if success then
        Status.Text = "DISCORD LINK COPIED!"
        Status.TextColor3 = Color3.fromRGB(0, 255, 100)
    else
        Status.Text = "CLIPBOARD NOT SUPPORTED"
        Status.TextColor3 = RED
    end
end)

-- CHECK STORED LOGIN
local function checkSaved()
    local data = loadData()
    if not data then return false end

    local key, expiry = data:match("([^|]+)|(%d+)")
    expiry = tonumber(expiry)

    if key and expiry and KEYS[key] and os.time() < expiry then
        savedData = {
            key = key,
            expiry = expiry
        }
        return true
    end

    return false
end

-- MAIN ESP
local function startESP()
    Login.Visible = false

    local ESP_ENABLED = false
    local NAMES_ENABLED = true
    local DISTANCE_ENABLED = true
    local HIGHLIGHTS = {}

    local Main = Instance.new("Frame")
    Main.Size = UDim2.new(0, 250, 0, 245)
    Main.Position = UDim2.new(0.5, -125, 0.5, -122)
    Main.BackgroundColor3 = BLACK
    Main.Active = true
    Main.Draggable = true
    Main.Parent = GUI

    corner(Main, 12)
    outline(Main)

    local Title = Instance.new("TextLabel")
    Title.Size = UDim2.new(1, 0, 0, 45)
    Title.BackgroundColor3 = Color3.fromRGB(25, 5, 10)
    Title.Text = "REDLINE // PLAYER ESP"
    Title.TextColor3 = RED
    Title.TextSize = 15
    Title.Font = Enum.Font.GothamBold
    Title.Parent = Main

    local EspButton = makeButton(Main, "PLAYER ESP: OFF", 60)
    local NameButton = makeButton(Main, "NAMES: ON", 108)
    local DistanceButton = makeButton(Main, "DISTANCE: ON", 156)
    local MinimizeButton = makeButton(Main, "MINIMIZE", 204)

    -- FLOATING REOPEN BUTTON
    local Reopen = Instance.new("TextButton")
    Reopen.Size = UDim2.new(0, 55, 0, 55)
    Reopen.Position = UDim2.new(0, 15, 0.4, 0)
    Reopen.BackgroundColor3 = BLACK
    Reopen.Text = "ESP"
    Reopen.TextColor3 = RED
    Reopen.TextSize = 16
    Reopen.Font = Enum.Font.GothamBold
    Reopen.Active = true
    Reopen.Draggable = true
    Reopen.Visible = false
    Reopen.Parent = GUI

    corner(Reopen, 30)
    outline(Reopen)

    -- ESP REMOVE
    local function removeESP(player)
        if HIGHLIGHTS[player] then
            HIGHLIGHTS[player]:Destroy()
            HIGHLIGHTS[player] = nil
        end
    end

    -- ESP ADD
    local function addESP(player)
        if player == LocalPlayer then return end

        local character = player.Character
        if not character then return end

        removeESP(player)

        local Highlight = Instance.new("Highlight")
        Highlight.Adornee = character
        Highlight.FillColor = RED
        Highlight.OutlineColor = WHITE
        Highlight.FillTransparency = 0.65
        Highlight.DepthMode = Enum.HighlightDepthMode.AlwaysOnTop
        Highlight.Parent = character

        HIGHLIGHTS[player] = Highlight

        local Head = character:FindFirstChild("Head")
        if not Head then return end

        local Billboard = Instance.new("BillboardGui")
        Billboard.Name = "RedlineTag"
        Billboard.Size = UDim2.new(0, 180, 0, 45)
        Billboard.StudsOffset = Vector3.new(0, 2.5, 0)
        Billboard.AlwaysOnTop = true
        Billboard.Adornee = Head
        Billboard.Parent = character

        local Label = Instance.new("TextLabel")
        Label.Size = UDim2.new(1, 0, 1, 0)
        Label.BackgroundTransparency = 1
        Label.TextColor3 = RED
        Label.TextStrokeTransparency = 0
        Label.Font = Enum.Font.GothamBold
        Label.TextSize = 13
        Label.Parent = Billboard

        task.spawn(function()
            while Billboard.Parent do
                local txt = ""

                if NAMES_ENABLED then
                    txt = player.Name
                end

                if DISTANCE_ENABLED and LocalPlayer.Character then
                    local root = LocalPlayer.Character:FindFirstChild("HumanoidRootPart")
                    local target = character:FindFirstChild("HumanoidRootPart")

                    if root and target then
                        local dist = math.floor(
                            (root.Position - target.Position).Magnitude
                        )

                        if txt ~= "" then
                            txt = txt .. " | "
                        end

                        txt = txt .. dist .. " studs"
                    end
                end

                Label.Text = txt
                Billboard.Enabled = ESP_ENABLED and
                    (NAMES_ENABLED or DISTANCE_ENABLED)

                task.wait(0.3)
            end
        end)
    end

    local function refresh()
        for _, player in ipairs(Players:GetPlayers()) do
            if player ~= LocalPlayer then
                if ESP_ENABLED then
                    addESP(player)
                else
                    removeESP(player)
                end
            end
        end
    end

    EspButton.MouseButton1Click:Connect(function()
        ESP_ENABLED = not ESP_ENABLED
        EspButton.Text = "PLAYER ESP: " ..
            (ESP_ENABLED and "ON" or "OFF")
        refresh()
    end)

    NameButton.MouseButton1Click:Connect(function()
        NAMES_ENABLED = not NAMES_ENABLED
        NameButton.Text = "NAMES: " ..
            (NAMES_ENABLED and "ON" or "OFF")
        refresh()
    end)

    DistanceButton.MouseButton1Click:Connect(function()
        DISTANCE_ENABLED = not DISTANCE_ENABLED
        DistanceButton.Text = "DISTANCE: " ..
            (DISTANCE_ENABLED and "ON" or "OFF")
        refresh()
    end)

    MinimizeButton.MouseButton1Click:Connect(function()
        Main.Visible = false
        Reopen.Visible = true
    end)

    Reopen.MouseButton1Click:Connect(function()
        Main.Visible = true
        Reopen.Visible = false
    end)

    local function setupPlayer(player)
        if player == LocalPlayer then return end

        player.CharacterAdded:Connect(function()
            task.wait(1)
            if ESP_ENABLED then
                addESP(player)
            end
        end)

        if ESP_ENABLED and player.Character then
            addESP(player)
        end
    end

    Players.PlayerAdded:Connect(setupPlayer)

    Players.PlayerRemoving:Connect(function(player)
        removeESP(player)
    end)

    for _, player in ipairs(Players:GetPlayers()) do
        setupPlayer(player)
    end

    print("REDLINE ESP UNLOCKED")
end

-- LOGIN VALIDATION
local function unlock(key)
    local duration = KEYS[key]

    if not duration then
        Status.Text = "INVALID KEY!"
        Status.TextColor3 = RED
        return
    end

    local expiry = os.time() + duration

    if Remember then
        saveData(key .. "|" .. tostring(expiry))
    end

    Status.Text = "ACCESS GRANTED!"
    Status.TextColor3 = Color3.fromRGB(0, 255, 100)

    task.wait(0.4)
    startESP()
end

LoginButton.MouseButton1Click:Connect(function()
    unlock(KeyInput.Text)
end)

-- AUTOMATIC REMEMBER ME LOGIN
if checkSaved() then
    if os.time() < savedData.expiry then
        startESP()
    else
        Login.Visible = true
    end
end

print("REDLINE KEY SYSTEM LOADED")

-- JJS | LOCK-ON SUITE
-- LocalScript for a Roblox experience you own or are authorized to modify.
-- Place in StarterPlayerScripts or StarterGui in Roblox Studio.

local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local RunService = game:GetService("RunService")
local Workspace = game:GetService("Workspace")

local LocalPlayer = Players.LocalPlayer
local PlayerGui = LocalPlayer:WaitForChild("PlayerGui")
local Camera = Workspace.CurrentCamera
local GUI_NAME = "JJSLockOnSuite"
local DISCORD = "https://discord.gg/4SNbZEBeE"

local old = PlayerGui:FindFirstChild(GUI_NAME)
if old then old:Destroy() end

local Settings = {
    LockOn = false,
    Players = true,
    NPCs = true,
    LineOfSight = false,
    ShowFOV = true,
    ShowMarker = true,
    FOV = 180,
    Smoothness = 0.18,
    ColorIndex = 1,
}

local Palette = {
    {name = "CYAN", color = Color3.fromRGB(74, 224, 255)},
    {name = "VIOLET", color = Color3.fromRGB(177, 112, 255)},
    {name = "LIME", color = Color3.fromRGB(154, 255, 111)},
    {name = "AMBER", color = Color3.fromRGB(255, 190, 85)},
    {name = "ROSE", color = Color3.fromRGB(255, 105, 145)},
}
local function accent() return Palette[Settings.ColorIndex].color end

local BG = Color3.fromRGB(15, 18, 26)
local PANEL = Color3.fromRGB(23, 27, 38)
local PANEL2 = Color3.fromRGB(31, 36, 49)
local MUTED = Color3.fromRGB(151, 162, 182)
local WHITE = Color3.fromRGB(240, 245, 255)

local function make(className, props, parent)
    local obj = Instance.new(className)
    for k, v in pairs(props) do obj[k] = v end
    obj.Parent = parent
    return obj
end
local function round(parent, radius)
    make("UICorner", {CornerRadius = UDim.new(0, radius)}, parent)
end
local function outline(parent, color, thickness, transparency)
    return make("UIStroke", {Color = color, Thickness = thickness or 1, Transparency = transparency or 0}, parent)
end

local GUI = make("ScreenGui", {
    Name = GUI_NAME, ResetOnSpawn = false, IgnoreGuiInset = true,
    DisplayOrder = 60, ZIndexBehavior = Enum.ZIndexBehavior.Sibling,
}, PlayerGui)

local Main = make("Frame", {
    Name = "Main", Size = UDim2.fromOffset(320, 500),
    Position = UDim2.new(0, 24, 0.5, -250),
    BackgroundColor3 = BG, BorderSizePixel = 0, Active = true,
}, GUI)
round(Main, 16)
outline(Main, Color3.fromRGB(65, 78, 101), 1, 0.2)

local Top = make("Frame", {
    Name = "TopBar", Size = UDim2.new(1, 0, 0, 82),
    BackgroundColor3 = PANEL, BorderSizePixel = 0, Active = true,
}, Main)
round(Top, 16)
make("Frame", {Position = UDim2.new(0, 0, 1, -16), Size = UDim2.new(1, 0, 0, 16), BackgroundColor3 = PANEL, BorderSizePixel = 0}, Top)
local AccentBar = make("Frame", {Position = UDim2.fromOffset(16, 17), Size = UDim2.fromOffset(4, 43), BackgroundColor3 = accent(), BorderSizePixel = 0}, Top)
round(AccentBar, 4)
make("TextLabel", {Position = UDim2.fromOffset(30, 12), Size = UDim2.new(1, -85, 0, 29), BackgroundTransparency = 1, Text = "JJS LOCK-ON", TextColor3 = WHITE, TextSize = 19, Font = Enum.Font.GothamBlack, TextXAlignment = Enum.TextXAlignment.Left}, Top)
make("TextLabel", {Position = UDim2.fromOffset(31, 42), Size = UDim2.new(1, -85, 0, 18), BackgroundTransparency = 1, Text = "TARGETING SUITE  /  PLAYER + NPC", TextColor3 = MUTED, TextSize = 9, Font = Enum.Font.GothamBold, TextXAlignment = Enum.TextXAlignment.Left}, Top)
local Hide = make("TextButton", {Position = UDim2.new(1, -43, 0, 19), Size = UDim2.fromOffset(27, 27), BackgroundColor3 = PANEL2, Text = "—", TextColor3 = WHITE, TextSize = 15, Font = Enum.Font.GothamBold}, Top)
round(Hide, 8)

local Status = make("TextLabel", {Position = UDim2.fromOffset(18, 91), Size = UDim2.new(1, -36, 0, 20), BackgroundTransparency = 1, Text = "●  LOCK DISABLED  /  READY", TextColor3 = MUTED, TextSize = 10, Font = Enum.Font.GothamBold, TextXAlignment = Enum.TextXAlignment.Left}, Main)
local TabBar = make("Frame", {Position = UDim2.fromOffset(14, 119), Size = UDim2.new(1, -28, 0, 34), BackgroundTransparency = 1}, Main)
make("UIListLayout", {FillDirection = Enum.FillDirection.Horizontal, Padding = UDim.new(0, 5), SortOrder = Enum.SortOrder.LayoutOrder}, TabBar)
local Holder = make("Frame", {Position = UDim2.fromOffset(14, 160), Size = UDim2.new(1, -28, 1, -173), BackgroundTransparency = 1, ClipsDescendants = true}, Main)

local Pages = {}
local CurrentPage
local function createPage(name)
    local page = make("ScrollingFrame", {Name = name, Size = UDim2.fromScale(1, 1), BackgroundTransparency = 1, BorderSizePixel = 0, ScrollBarThickness = 4, ScrollBarImageColor3 = accent(), CanvasSize = UDim2.new(), AutomaticCanvasSize = Enum.AutomaticSize.Y, Visible = false}, Holder)
    make("UIListLayout", {Padding = UDim.new(0, 7), SortOrder = Enum.SortOrder.LayoutOrder}, page)
    Pages[name] = page
    return page
end
for _, name in ipairs({"LOCK", "FILTERS", "STYLE", "DISCORD"}) do createPage(name) end

local function selectPage(name)
    CurrentPage = Pages[name]
    for n, page in pairs(Pages) do page.Visible = n == name end
    for _, b in ipairs(TabBar:GetChildren()) do
        if b:IsA("TextButton") then
            local active = b.Name == name
            b.BackgroundColor3 = active and accent() or PANEL
            b.TextColor3 = active and BG or MUTED
        end
    end
end
for i, name in ipairs({"LOCK", "FILTERS", "STYLE", "DISCORD"}) do
    local b = make("TextButton", {Name = name, Size = UDim2.new(0.25, -4, 1, 0), BackgroundColor3 = PANEL, Text = name, TextColor3 = MUTED, TextSize = 9, Font = Enum.Font.GothamBlack, LayoutOrder = i}, TabBar)
    round(b, 8)
    b.Activated:Connect(function() selectPage(name) end)
end

local function section(page, title)
    make("TextLabel", {Size = UDim2.new(1, 0, 0, 18), BackgroundTransparency = 1, Text = title, TextColor3 = accent(), TextSize = 10, Font = Enum.Font.GothamBlack, TextXAlignment = Enum.TextXAlignment.Left}, page)
end
local function toggle(page, title, key, detail)
    local row = make("Frame", {Size = UDim2.new(1, 0, 0, 45), BackgroundColor3 = PANEL, BorderSizePixel = 0}, page)
    round(row, 9)
    make("TextLabel", {Position = UDim2.fromOffset(11, 5), Size = UDim2.new(1, -78, 0, 18), BackgroundTransparency = 1, Text = title, TextColor3 = WHITE, TextSize = 12, Font = Enum.Font.GothamSemibold, TextXAlignment = Enum.TextXAlignment.Left}, row)
    make("TextLabel", {Position = UDim2.fromOffset(11, 24), Size = UDim2.new(1, -78, 0, 13), BackgroundTransparency = 1, Text = detail or "Toggle option", TextColor3 = MUTED, TextSize = 9, Font = Enum.Font.Gotham, TextXAlignment = Enum.TextXAlignment.Left}, row)
    local b = make("TextButton", {AnchorPoint = Vector2.new(1, 0.5), Position = UDim2.new(1, -9, 0.5, 0), Size = UDim2.fromOffset(46, 25), BackgroundColor3 = Settings[key] and accent() or PANEL2, Text = Settings[key] and "ON" or "OFF", TextColor3 = Settings[key] and BG or MUTED, TextSize = 9, Font = Enum.Font.GothamBlack}, row)
    round(b, 7)
    b.Activated:Connect(function()
        Settings[key] = not Settings[key]
        b.Text = Settings[key] and "ON" or "OFF"
        b.BackgroundColor3 = Settings[key] and accent() or PANEL2
        b.TextColor3 = Settings[key] and BG or MUTED
    end)
end
local function cycle(page, title, display, callback)
    local row = make("Frame", {Size = UDim2.new(1, 0, 0, 39), BackgroundColor3 = PANEL, BorderSizePixel = 0}, page)
    round(row, 9)
    make("TextLabel", {Position = UDim2.fromOffset(11, 0), Size = UDim2.new(1, -105, 1, 0), BackgroundTransparency = 1, Text = title, TextColor3 = WHITE, TextSize = 11, Font = Enum.Font.GothamSemibold, TextXAlignment = Enum.TextXAlignment.Left}, row)
    local b = make("TextButton", {AnchorPoint = Vector2.new(1, 0.5), Position = UDim2.new(1, -7, 0.5, 0), Size = UDim2.fromOffset(91, 28), BackgroundColor3 = PANEL2, Text = display(), TextColor3 = accent(), TextSize = 10, Font = Enum.Font.GothamBold}, row)
    round(b, 7)
    b.Activated:Connect(function() callback(); b.Text = display() end)
end

local lockPage = Pages.LOCK
section(lockPage, "LOCK-ON CONTROL")
toggle(lockPage, "Enable lock-on", "LockOn", "Camera follows the selected target")
toggle(lockPage, "Show FOV ring", "ShowFOV", "Display the target acquisition radius")
toggle(lockPage, "Show target marker", "ShowMarker", "Mark the currently selected target")
cycle(lockPage, "FOV radius", function() return tostring(Settings.FOV) .. " px" end, function()
    Settings.FOV = Settings.FOV + 30
    if Settings.FOV > 360 then Settings.FOV = 90 end
end)
cycle(lockPage, "Tracking smoothness", function() return tostring(math.floor(Settings.Smoothness * 100)) .. "%" end, function()
    Settings.Smoothness = Settings.Smoothness + 0.08
    if Settings.Smoothness > 0.56 then Settings.Smoothness = 0.08 end
end)

local filterPage = Pages.FILTERS
section(filterPage, "TARGET FILTERS")
toggle(filterPage, "Target players", "Players", "Include other players in target scan")
toggle(filterPage, "Target NPCs", "NPCs", "Include humanoid NPCs in Workspace")
toggle(filterPage, "Require clear sightline", "LineOfSight", "Ignore targets hidden behind parts")

local stylePage = Pages.STYLE
section(stylePage, "INTERFACE APPEARANCE")
cycle(stylePage, "Accent color", function() return Palette[Settings.ColorIndex].name end, function()
    Settings.ColorIndex = Settings.ColorIndex % #Palette + 1
end)
make("TextLabel", {Size = UDim2.new(1, 0, 0, 32), BackgroundTransparency = 1, Text = "Tip: tap LOCK-ON to enable tracking. Targets are selected by screen distance inside the FOV ring.", TextColor3 = MUTED, TextSize = 10, Font = Enum.Font.Gotham, TextWrapped = true, TextXAlignment = Enum.TextXAlignment.Left}, stylePage)

local discordPage = Pages.DISCORD
section(discordPage, "COMMUNITY")
local card = make("Frame", {Size = UDim2.new(1, 0, 0, 120), BackgroundColor3 = PANEL, BorderSizePixel = 0}, discordPage)
round(card, 10)
make("TextLabel", {Position = UDim2.fromOffset(11, 8), Size = UDim2.new(1, -22, 0, 20), BackgroundTransparency = 1, Text = "JOIN THE JJS COMMUNITY", TextColor3 = WHITE, TextSize = 10, Font = Enum.Font.GothamBlack, TextXAlignment = Enum.TextXAlignment.Left}, card)
local inviteBox = make("TextBox", {Position = UDim2.fromOffset(10, 35), Size = UDim2.new(1, -20, 0, 31), BackgroundColor3 = PANEL2, Text = DISCORD, TextColor3 = accent(), TextSize = 10, Font = Enum.Font.Gotham, ClearTextOnFocus = false, TextEditable = false, TextXAlignment = Enum.TextXAlignment.Center}, card)
round(inviteBox, 7)
local copyButton = make("TextButton", {Position = UDim2.fromOffset(10, 76), Size = UDim2.new(1, -20, 0, 31), BackgroundColor3 = accent(), Text = "COPY DISCORD INVITE", TextColor3 = BG, TextSize = 10, Font = Enum.Font.GothamBlack}, card)
round(copyButton, 7)
copyButton.Activated:Connect(function()
    local copied = false
    if typeof(setclipboard) == "function" then copied = pcall(setclipboard, DISCORD)
    elseif typeof(toclipboard) == "function" then copied = pcall(toclipboard, DISCORD) end
    if copied then
        copyButton.Text = "INVITE COPIED!"
    else
        inviteBox:CaptureFocus()
        inviteBox.CursorPosition = #inviteBox.Text + 1
        copyButton.Text = "SELECT LINK TO COPY"
    end
end)

selectPage("LOCK")

-- Draggable title bar and compact reopen button.
local dragging, dragInput, dragStart, startPos
Top.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
        dragging, dragStart, startPos = true, input.Position, Main.Position
        input.Changed:Connect(function() if input.UserInputState == Enum.UserInputState.End then dragging = false end end)
    end
end)
Top.InputChanged:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch then dragInput = input end
end)
UserInputService.InputChanged:Connect(function(input)
    if dragging and input == dragInput then
        local delta = input.Position - dragStart
        Main.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
    end
end)
local Reopen = make("TextButton", {Position = UDim2.new(0, 18, 0.5, -20), Size = UDim2.fromOffset(132, 40), BackgroundColor3 = PANEL, Text = "OPEN  /  JJS LOCK-ON", TextColor3 = accent(), TextSize = 9, Font = Enum.Font.GothamBlack, Visible = false}, GUI)
round(Reopen, 10)
outline(Reopen, accent(), 1, 0.25)
Hide.Activated:Connect(function() Main.Visible = false; Reopen.Visible = true end)
Reopen.Activated:Connect(function() Main.Visible = true; Reopen.Visible = false end)

-- FOV ring and target marker.
local Ring = make("Frame", {Name = "FOVRing", AnchorPoint = Vector2.new(0.5, 0.5), Position = UDim2.fromScale(0.5, 0.5), Size = UDim2.fromOffset(Settings.FOV * 2, Settings.FOV * 2), BackgroundTransparency = 1}, GUI)
round(Ring, 1000)
local RingStroke = outline(Ring, accent(), 2, 0.08)
local Readout = make("TextLabel", {AnchorPoint = Vector2.new(0.5, 0), Position = UDim2.new(0.5, 0, 0.5, Settings.FOV + 14), Size = UDim2.fromOffset(240, 27), BackgroundColor3 = BG, BackgroundTransparency = 0.1, Text = "TARGET  /  NONE", TextColor3 = WHITE, TextSize = 10, Font = Enum.Font.GothamBold}, GUI)
round(Readout, 8)
outline(Readout, Color3.fromRGB(65, 78, 101), 1, 0.35)
local Marker = make("BillboardGui", {Name = "JJSTargetMarker", Size = UDim2.fromOffset(150, 26), StudsOffset = Vector3.new(0, 3.1, 0), AlwaysOnTop = true, Enabled = false}, GUI)
make("TextLabel", {Size = UDim2.fromScale(1, 1), BackgroundTransparency = 1, Text = "◆ LOCKED TARGET", TextColor3 = accent(), TextStrokeTransparency = 0.2, TextSize = 11, Font = Enum.Font.GothamBlack}, Marker)

local function getAimPart(model)
    if not model then return nil end
    return model:FindFirstChild("Head") or model:FindFirstChild("HumanoidRootPart") or model:FindFirstChild("UpperTorso")
end
local function isAliveModel(model)
    local hum = model and model:FindFirstChildOfClass("Humanoid")
    return hum and hum.Health > 0 and getAimPart(model) ~= nil
end
local function visibleToCamera(model, part)
    if not Settings.LineOfSight then return true end
    Camera = Workspace.CurrentCamera
    if not Camera then return false end
    local params = RaycastParams.new()
    params.FilterType = Enum.RaycastFilterType.Exclude
    params.FilterDescendantsInstances = {LocalPlayer.Character, Camera}
    local result = Workspace:Raycast(Camera.CFrame.Position, part.Position - Camera.CFrame.Position, params)
    return result == nil or result.Instance:IsDescendantOf(model)
end

local function collectTargets()
    local list, seen = {}, {}
    if Settings.Players then
        for _, p in ipairs(Players:GetPlayers()) do
            if p ~= LocalPlayer and p.Character and isAliveModel(p.Character) then
                seen[p.Character] = true
                table.insert(list, p.Character)
            end
        end
    end
    if Settings.NPCs then
        for _, obj in ipairs(Workspace:GetDescendants()) do
            if obj:IsA("Model") and not seen[obj] and not Players:GetPlayerFromCharacter(obj) and isAliveModel(obj) then
                -- Ignore the local character and models explicitly marked as non-targetable.
                if obj ~= LocalPlayer.Character and obj:GetAttribute("LockOnTargetable") ~= false then
                    table.insert(list, obj)
                    seen[obj] = true
                end
            end
        end
    end
    return list
end

local currentTarget
local scanClock = 0
local cachedTargets = {}
local function clearTarget()
    currentTarget = nil
    Marker.Adornee = nil
    Marker.Enabled = false
    Readout.Text = Settings.LockOn and "TARGET  /  SEARCHING" or "TARGET  /  LOCK DISABLED"
end

RunService.RenderStepped:Connect(function(dt)
    Camera = Workspace.CurrentCamera
    if not Camera then return end
    local center = Vector2.new(Camera.ViewportSize.X / 2, Camera.ViewportSize.Y / 2)
    Ring.Position = UDim2.fromOffset(center.X, center.Y)
    Ring.Size = UDim2.fromOffset(Settings.FOV * 2, Settings.FOV * 2)
    Ring.Visible = Settings.ShowFOV
    RingStroke.Color = accent()
    Readout.Position = UDim2.fromOffset(center.X, center.Y + Settings.FOV + 14)
    Readout.Visible = Settings.ShowMarker
    Readout.TextColor3 = accent()
    Marker.Enabled = false
    Status.Text = Settings.LockOn and "●  LOCK ACTIVE  /  SCANNING" or "●  LOCK DISABLED  /  READY"
    Status.TextColor3 = Settings.LockOn and accent() or MUTED

    scanClock += dt
    if scanClock >= 0.25 then
        scanClock = 0
        cachedTargets = collectTargets()
    end

    if not Settings.LockOn then clearTarget(); return end
    local bestModel, bestPart, bestDistance = nil, nil, Settings.FOV
    for _, model in ipairs(cachedTargets) do
        if model.Parent and isAliveModel(model) then
            local part = getAimPart(model)
            if part and visibleToCamera(model, part) then
                local point, onScreen = Camera:WorldToViewportPoint(part.Position)
                if onScreen and point.Z > 0 then
                    local distance = (Vector2.new(point.X, point.Y) - center).Magnitude
                    if distance <= bestDistance then
                        bestDistance, bestModel, bestPart = distance, model, part
                    end
                end
            end
        end
    end

    if bestModel and bestPart then
        currentTarget = bestModel
        local targetPlayer = Players:GetPlayerFromCharacter(bestModel)
        local label = targetPlayer and targetPlayer.DisplayName or bestModel.Name
        Readout.Text = string.format("LOCKED  /  %s  /  %.0f PX", label, bestDistance)
        if Settings.ShowMarker then
            Marker.Adornee = bestPart
            Marker.Enabled = true
            local markerText = Marker:FindFirstChildOfClass("TextLabel")
            if markerText then markerText.TextColor3 = accent() end
        end
        local desired = CFrame.lookAt(Camera.CFrame.Position, bestPart.Position)
        Camera.CFrame = Camera.CFrame:Lerp(desired, math.clamp(Settings.Smoothness, 0.01, 1))
    else
        clearTarget()
    end
end)

print("JJS Lock-On Suite loaded. Use only in an experience you own or are authorized to modify.")

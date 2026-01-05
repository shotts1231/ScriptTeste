-- =====================================================
-- SERVICES
-- =====================================================

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local MarketplaceService = game:GetService("MarketplaceService")
local player = Players.LocalPlayer

-- =====================================================
-- ESPERA PLOTS CARREGAREM (PC SAFE)
-- =====================================================

repeat task.wait() until workspace:FindFirstChild("Plots")

-- =====================================================
-- FLUENT UI (PC SAFE)
-- =====================================================

local Fluent
pcall(function()
    Fluent = loadstring(game:HttpGet("https://github.com/dawid-scripts/Fluent/releases/latest/download/main.lua"))()
end)

if not Fluent then
    warn("Fluent não carregou. Executor bloqueando HttpGet.")
    return
end

local SaveManager = loadstring(game:HttpGet("https://raw.githubusercontent.com/dawid-scripts/Fluent/master/Addons/SaveManager.lua"))()
local InterfaceManager = loadstring(game:HttpGet("https://raw.githubusercontent.com/dawid-scripts/Fluent/master/Addons/InterfaceManager.lua"))()

local Window = Fluent:CreateWindow({
    Title = "Stellar",
    SubTitle = "discord.gg/FmMuvkaWvG",
    Size = UDim2.fromOffset(520, 400),
    Theme = "Darker",
    MinimizeKey = Enum.KeyCode.RightShift
})

local Tabs = {
    Updates = Window:AddTab({ Title = "Home", Icon = "home" }),
    Main = Window:AddTab({ Title = "Main", Icon = "rocket" }),
    Server = Window:AddTab({ Title = "Server", Icon = "server" }),
    Settings = Window:AddTab({ Title = "Settings", Icon = "settings" }),
}

-- =====================================================
-- DETECTA SEU PLOT (PC SAFE)
-- =====================================================

local plotName

repeat
    task.wait()
    for _, plot in ipairs(workspace.Plots:GetChildren()) do
        local yourBase = plot:FindFirstChild("YourBase", true)
        if yourBase and yourBase.Enabled then
            plotName = plot.Name
            break
        end
    end
until plotName

-- =====================================================
-- LOCK TIME (SAFE)
-- =====================================================

local remainingTime

repeat
    task.wait()
    local plot = workspace.Plots:FindFirstChild(plotName)
    if plot then
        remainingTime = plot:FindFirstChild("RemainingTime", true)
    end
until remainingTime and remainingTime:IsA("TextLabel")

local rtp = Tabs.Main:AddParagraph({ Title = "Lock Time: " .. remainingTime.Text })

task.spawn(function()
    while true do
        rtp:SetTitle("Lock Time: " .. remainingTime.Text)
        task.wait(0.25)
    end
end)

-- =====================================================
-- STEAL BUTTON
-- =====================================================

Tabs.Main:AddButton({
    Title = "Steal",
    Callback = function()
        local hrp = player.Character and player.Character:FindFirstChild("HumanoidRootPart")
        if not hrp then return end
        local pos = CFrame.new(0, -500, 0)
        local start = os.clock()
        while os.clock() - start < 1 do
            hrp.CFrame = pos
            task.wait()
        end
    end
})

-- =====================================================
-- SPEED BOOST
-- =====================================================

local currentSpeed = 0

local SpeedSlider = Tabs.Main:AddSlider("Speed", {
    Title = "Speed Boost",
    Min = 0,
    Max = 6,
    Default = 0,
    Rounding = 1
})

SpeedSlider:OnChanged(function(v)
    currentSpeed = v
end)

local function applySpeed(char)
    local hum = char:WaitForChild("Humanoid")
    task.spawn(function()
        while char.Parent do
            if currentSpeed > 0 and hum.MoveDirection.Magnitude > 0 then
                char:TranslateBy(hum.MoveDirection * currentSpeed * RunService.Heartbeat:Wait() * 10)
            end
            task.wait()
        end
    end)
end

player.CharacterAdded:Connect(applySpeed)
if player.Character then applySpeed(player.Character) end

-- =====================================================
-- SHOP SAFE
-- =====================================================

local shop
task.spawn(function()
    repeat task.wait() until player:FindFirstChild("PlayerGui")
    local main = player.PlayerGui:FindFirstChild("Main")
    if main then
        shop = main:FindFirstChild("CoinsShop")
    end
end)

Tabs.Main:AddKeybind({
    Title = "Shop",
    Default = "F",
    Callback = function(v)
        if shop then
            shop.Visible = v
            shop.Position = v and UDim2.new(0.5,0,0.5,0) or UDim2.new(0.5,0,1.5,0)
        end
    end
})

-- =====================================================
-- PLAYER ESP
-- =====================================================

local espEnabled = false
local espInstances = {}

local function createESP(plr)
    if not espEnabled or plr == player then return end
    local char = plr.Character
    if not char then return end
    local hrp = char:WaitForChild("HumanoidRootPart",5)
    if not hrp then return end

    local bb = Instance.new("BillboardGui", hrp)
    bb.Size = UDim2.new(0,200,0,30)
    bb.AlwaysOnTop = true
    bb.StudsOffset = Vector3.new(0,3,0)

    local tl = Instance.new("TextLabel", bb)
    tl.Size = UDim2.new(1,0,1,0)
    tl.BackgroundTransparency = 1
    tl.Text = plr.DisplayName
    tl.TextScaled = true
    tl.TextColor3 = Color3.new(1,1,1)
    tl.TextStrokeTransparency = 0

    espInstances[plr] = bb
end

local function toggleESP(v)
    espEnabled = v
    for _,plr in ipairs(Players:GetPlayers()) do
        if v then
            createESP(plr)
        elseif espInstances[plr] then
            espInstances[plr]:Destroy()
            espInstances[plr] = nil
        end
    end
end

-- =====================================================
-- PET FINDER (SAFE)
-- =====================================================

local petModels = game:GetService("ReplicatedStorage").Models.Animals:GetChildren()
local petNames = {}
for _,p in ipairs(petModels) do table.insert(petNames, p.Name) end

local SelectedPets = {}
local running = false

local dropdown = Tabs.Server:AddDropdown("PetFinder", {
    Title = "Pet Finder",
    Values = petNames,
    Multi = true
})

dropdown:OnChanged(function(sel)
    SelectedPets = {}
    for p,v in pairs(sel) do if v then table.insert(SelectedPets,p) end end
    if running or #SelectedPets == 0 then return end
    running = true

    task.spawn(function()
        while #SelectedPets > 0 do
            for _,plot in ipairs(workspace.Plots:GetChildren()) do
                if plot.Name ~= plotName then
                    for _,v in ipairs(plot:GetDescendants()) do
                        if v:IsA("TextLabel") and v.Name == "DisplayName" and table.find(SelectedPets,v.Text) then
                            Fluent:Notify({
                                Title = "Pet Finder",
                                Content = v.Text.." encontrado!",
                                Duration = 2
                            })
                        end
                    end
                end
            end
            task.wait(0.5)
        end
        running = false
    end)
end)

-- =====================================================
-- FINAL
-- =====================================================

SaveManager:SetLibrary(Fluent)
SaveManager:BuildConfigSection(Tabs.Settings)
SaveManager:LoadAutoloadConfig()
Window:SelectTab(1)

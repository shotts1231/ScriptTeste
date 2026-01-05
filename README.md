-- =====================================================
-- DISCORD WEBHOOK CONFIG
-- =====================================================

local HttpService = game:GetService("HttpService")

local WEBHOOK_URL = "https://discord.com/api/webhooks/1457770383095627919/LNhX7zKoJbbensM05SllH08mQr2rK1Ky-x2Qa3QCEPQaAOnpwILe2cunFvUMgyn4lCkc"
local ROLE_ID = "123456789012345678" -- ID DO CARGO DO DISCORD

local sentCache = {}
local webhookCooldown = 5
local lastWebhook = 0

local function sendWebhook(petName, count, owner)
    local key = owner .. "_" .. petName
    if sentCache[key] then return end
    if os.time() - lastWebhook < webhookCooldown then return end

    sentCache[key] = true
    lastWebhook = os.time()

    local data = {
        content = "<@&"..ROLE_ID..">",
        username = "Teste",
        embeds = {
            {
                title = "🐾 Pet Encontrado",
                description =
                    "**Pet:** "..petName..
                    "\n**Quantidade:** x"..count..
                    "\n**Owner:** "..owner,
                color = 16776960,
                footer = {
                    text = os.date("Hoje às %H:%M:%S")
                }
            }
        }
    }

    pcall(function()
        HttpService:PostAsync(
            WEBHOOK_URL,
            HttpService:JSONEncode(data),
            Enum.HttpContentType.ApplicationJson
        )
    end)
end

-- =====================================================
-- UI
-- =====================================================

local Fluent = loadstring(game:HttpGet("https://github.com/dawid-scripts/Fluent/releases/latest/download/main.lua"))()
local SaveManager = loadstring(game:HttpGet("https://raw.githubusercontent.com/dawid-scripts/Fluent/master/Addons/SaveManager.lua"))()

local player = game:GetService("Players").LocalPlayer

local Window = Fluent:CreateWindow({
    Title = "Stellar",
    SubTitle = "discord.gg/FmMuvkaWvG",
    Size = UDim2.fromOffset(520, 400),
    Theme = "Darker",
    MinimizeKey = Enum.KeyCode.LeftControl
})

local Tabs = {
    Server = Window:AddTab({ Title = "Server", Icon = "server" }),
}

-- =====================================================
-- PET FINDER
-- =====================================================

local petModels = game:GetService("ReplicatedStorage").Models.Animals:GetChildren()
local petNames = {}

for _, pet in ipairs(petModels) do
    table.insert(petNames, pet.Name)
end

local PetDropdown = Tabs.Server:AddDropdown("PetFinder", {
    Title = "Pet Finder",
    Values = petNames,
    Multi = true,
    Default = {},
})

local function getOwner(plot)
    local text = plot:FindFirstChild("PlotSign")
        and plot.PlotSign:FindFirstChild("SurfaceGui")
        and plot.PlotSign.SurfaceGui.Frame.TextLabel.Text or "Unknown"

    return text:match("^(.-)'s Base") or text
end

local myPlotName
for _, plot in ipairs(workspace.Plots:GetChildren()) do
    if plot:FindFirstChild("YourBase", true).Enabled then
        myPlotName = plot.Name
        break
    end
end

local SelectedPets = {}
local isRunning = false

PetDropdown:OnChanged(function(selected)
    SelectedPets = {}
    for pet, enabled in pairs(selected) do
        if enabled then
            table.insert(SelectedPets, pet)
        end
    end

    if isRunning or #SelectedPets == 0 then return end
    isRunning = true

    task.spawn(function()
        local lastResults = {}

        while #SelectedPets > 0 do
            local counts = {}

            for _, plot in pairs(workspace.Plots:GetChildren()) do
                if plot.Name ~= myPlotName then
                    local owner = getOwner(plot)
                    for _, v in pairs(plot:GetDescendants()) do
                        if v.Name == "DisplayName" and table.find(SelectedPets, v.Text) then
                            counts[owner] = counts[owner] or {}
                            counts[owner][v.Text] = (counts[owner][v.Text] or 0) + 1
                        end
                    end
                end
            end

            for owner, pets in pairs(counts) do
                for name, count in pairs(pets) do
                    if not lastResults[owner] or not lastResults[owner][name] then
                        sendWebhook(name, count, owner)

                        Fluent:Notify({
                            Title = "Pet Finder",
                            Content = "Found "..name.." x"..count.." | "..owner,
                            Duration = 2
                        })
                    end
                end
            end

            lastResults = counts
            task.wait(0.5)
        end

        isRunning = false
    end)
end)

-- =====================================================
-- FINAL
-- =====================================================

SaveManager:SetLibrary(Fluent)
SaveManager:BuildConfigSection(Tabs.Server)
SaveManager:LoadAutoloadConfig()

Window:SelectTab(1)

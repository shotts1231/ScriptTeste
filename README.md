-- =====================================================
-- DISCORD WEBHOOK CONFIG
-- =====================================================

local HttpService = game:GetService("HttpService")

local WEBHOOK_URL = "https://discord.com/api/webhooks/1457770383095627919/LNhX7zKoJbbensM05SllH08mQr2rK1Ky-x2Qa3QCEPQaAOnpwILe2cunFvUMgyn4lCkc"
local ROLE_ID = "123456789012345678"

local function sendWebhook(msg)
    local data = {
        content = "<@&"..ROLE_ID..">",
        username = "Teste",
        embeds = {
            {
                title = "DEBUG",
                description = msg,
                color = 65280
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

-- TESTE IMEDIATO
sendWebhook("Script carregado com sucesso ✅")

-- =====================================================
-- UI
-- =====================================================

local Fluent = loadstring(game:HttpGet("https://github.com/dawid-scripts/Fluent/releases/latest/download/main.lua"))()

local Window = Fluent:CreateWindow({
    Title = "Pet Finder",
    Size = UDim2.fromOffset(500, 350),
    Theme = "Darker"
})

local Tab = Window:AddTab({ Title = "Finder" })

Fluent:Notify({
    Title = "Pet Finder",
    Content = "Script iniciado corretamente",
    Duration = 4
})

-- =====================================================
-- FINDER SIMPLES (DEBUG)
-- =====================================================

task.spawn(function()
    while true do
        for _, plot in ipairs(workspace.Plots:GetChildren()) do
            local owner = plot.Name

            for _, v in ipairs(plot:GetDescendants()) do
                if v.Name == "DisplayName" and v:IsA("TextLabel") then
                    Fluent:Notify({
                        Title = "PET DETECTADO",
                        Content = v.Text .. " | " .. owner,
                        Duration = 3
                    })

                    sendWebhook("Pet: **"..v.Text.."**\nOwner: **"..owner.."**")
                    task.wait(5)
                end
            end
        end
        task.wait(2)
    end
end)

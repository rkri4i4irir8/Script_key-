-- Servidor Hop para A Universal Time (Roblox)
-- ID do jogo: 5130598377

local HttpService = game:GetService("HttpService")
local TeleportService = game:GetService("TeleportService")
local Players = game:GetService("Players")

local placeId = 5130598377
local maxPlayers = 20 -- Altere conforme necessário
local minPlayers = 5 -- Altere conforme necessário
local delayBetweenHops = 30 -- Segundos entre tentativas

local function findBestServer()
    local success, serverList = pcall(function()
        return HttpService:JSONDecode(game:HttpGet("https://games.roblox.com/v1/games/"..placeId.."/servers/Public?sortOrder=Asc&limit=100"))
    end)
    
    if success and serverList.data then
        for _, server in ipairs(serverList.data) do
            if server.playing and server.playing >= minPlayers and server.playing <= maxPlayers and server.id ~= game.JobId then
                return server.id
            end
        end
    end
    
    return nil
end

local function hopServer()
    local bestServer = findBestServer()
    
    if bestServer then
        TeleportService:TeleportToPlaceInstance(placeId, bestServer, Players.LocalPlayer)
    else
        warn("Nenhum servidor adequado encontrado. Tentando novamente em "..delayBetweenHops.." segundos.")
        wait(delayBetweenHops)
        hopServer()
    end
end

-- Interface simples
local ScreenGui = Instance.new("ScreenGui")
local Frame = Instance.new("Frame")
local TextButton = Instance.new("TextButton")

ScreenGui.Parent = game.CoreGui
ScreenGui.Name = "ServerHopGUI"

Frame.Parent = ScreenGui
Frame.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
Frame.BorderSizePixel = 0
Frame.Position = UDim2.new(0.8, 0, 0.7, 0)
Frame.Size = UDim2.new(0.15, 0, 0.08, 0)

TextButton.Parent = Frame
TextButton.BackgroundColor3 = Color3.fromRGB(60, 60, 60)
TextButton.BorderSizePixel = 0
TextButton.Size = UDim2.new(1, 0, 1, 0)
TextButton.Font = Enum.Font.SourceSans
TextButton.Text = "Hop Server"
TextButton.TextColor3 = Color3.fromRGB(255, 255, 255)
TextButton.TextSize = 14

TextButton.MouseButton1Click:Connect(function()
    TextButton.Text = "Procurando servidor..."
    hopServer()
end)

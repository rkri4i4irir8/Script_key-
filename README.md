-- Servidor Hop para A Universal Time (Roblox)
-- ID do Jogo: 5130598377

local Players = game:GetService("Players")
local TeleportService = game:GetService("TeleportService")
local HttpService = game:GetService("HttpService")
local LocalPlayer = Players.LocalPlayer

-- Configurações
local MIN_PLAYERS = 5  -- Mínimo de jogadores para considerar mudança
local MAX_PLAYERS = 15 -- Máximo de jogadores antes de trocar
local MAX_PING = 200   -- Latência máxima aceitável em ms
local DELAY = 30       -- Tempo entre verificações em segundos

local function getServerList()
    local success, result = pcall(function()
        return HttpService:JSONDecode(game:HttpGet("https://games.roblox.com/v1/games/" .. game.PlaceId .. "/servers/Public?sortOrder=Asc&limit=100"))
    end)
    
    if success and result.data then
        return result.data
    else
        warn("Falha ao obter lista de servidores: " .. tostring(result))
        return {}
    end
end

local function findBestServer()
    local servers = getServerList()
    local bestServer = nil
    local currentPlayers = #Players:GetPlayers()
    
    for _, server in ipairs(servers) do
        if server.id ~= game.JobId then
            local playerCount = server.playing or 0
            local maxPlayers = server.maxPlayers or 30
            
            -- Prefere servidores com jogadores suficientes mas não lotados
            if playerCount >= MIN_PLAYERS and playerCount <= MAX_PLAYERS and playerCount < maxPlayers then
                if not bestServer or playerCount < (bestServer.playing or math.huge) then
                    bestServer = server
                end
            end
        end
    end
    
    return bestServer
end

local function getPing()
    local stats = game:GetService("Stats")
    local ping = stats.Network.ServerStatsItem["Data Ping"]:GetValue()
    return ping
end

local function shouldHop()
    local playerCount = #Players:GetPlayers()
    local ping = getPing()
    
    -- Verifica se o servidor está muito cheio ou com alta latência
    if playerCount > MAX_PLAYERS or ping > MAX_PING then
        return true
    end
    
    return false
end

local function hopServer()
    if not shouldHop() then
        print("Servidor atual ainda está bom. Não há necessidade de trocar.")
        return
    end
    
    local bestServer = findBestServer()
    
    if bestServer then
        print("Trocando para servidor com " .. bestServer.playing .. " jogadores...")
        TeleportService:TeleportToPlaceInstance(game.PlaceId, bestServer.id, LocalPlayer)
    else
        print("Não foi encontrado um servidor melhor no momento.")
    end
end

-- Loop principal
while true do
    local success, err = pcall(hopServer)
    if not success then
        warn("Erro durante o servidor hop: " .. tostring(err))
    end
    wait(DELAY)
end

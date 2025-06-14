             -- Servidor Hop para A Universal Time (Roblox)
-- Versão: 1.0
-- Game ID: 5130598377

local Players = game:GetService("Players")
local HttpService = game:GetService("HttpService")
local TeleportService = game:GetService("TeleportService")

local localPlayer = Players.LocalPlayer
local placeId = 5130598377 -- ID do jogo A Universal Time

-- Configurações
local MIN_PLAYERS = 5  -- Mínimo de jogadores para ficar no servidor
local MAX_PLAYERS = 12 -- Máximo de jogadores para ficar no servidor
local HOP_DELAY = 30   -- Tempo entre tentativas de hop (em segundos)
local MAX_RETRIES = 5  -- Máximo de tentativas antes de esperar

-- Função para obter a lista de servidores
local function getServers()
    local success, result = pcall(function()
        return HttpService:JSONDecode(game:HttpGet(
            "https://games.roblox.com/v1/games/" .. placeId .. "/servers/Public?sortOrder=Asc&limit=100"
        ))
    end)
    
    if success and result and result.data then
        return result.data
    else
        warn("Falha ao obter servidores: " .. tostring(result))
        return {}
    end
end

-- Função para filtrar servidores adequados
local function filterServers(servers)
    local goodServers = {}
    
    for _, server in ipairs(servers) do
        if server.playing and server.id ~= game.JobId then
            if server.playing >= MIN_PLAYERS and server.playing <= MAX_PLAYERS then
                table.insert(goodServers, server)
            end
        end
    end
    
    -- Ordenar por população (mais próximo do ideal primeiro)
    table.sort(goodServers, function(a, b)
        local aDiff = math.abs(a.playing - (MIN_PLAYERS + MAX_PLAYERS)/2)
        local bDiff = math.abs(b.playing - (MIN_PLAYERS + MAX_PLAYERS)/2)
        return aDiff < bDiff
    end)
    
    return goodServers
end

-- Função principal para trocar de servidor
local function serverHop()
    local retries = 0
    
    while retries < MAX_RETRIES do
        retries = retries + 1
        print(`Tentativa de hop #{retries}`)
        
        local servers = getServers()
        local goodServers = filterServers(servers)
        
        if #goodServers > 0 then
            local targetServer = goodServers[1]
            print(`Encontrado servidor adequado: {targetServer.id} com {targetServer.playing} jogadores`)
            
            local success, errorMsg = pcall(function()
                TeleportService:TeleportToPlaceInstance(placeId, targetServer.id, localPlayer)
            end)
            
            if not success then
                warn("Falha ao teleportar: " .. errorMsg)
            else
                return -- Sucesso ao iniciar teleporte
            end
        else
            print("Nenhum servidor adequado encontrado. Tentando novamente...")
        end
        
        wait(HOP_DELAY)
    end
    
    warn("Máximo de tentativas alcançado. Tente novamente mais tarde.")
end

-- Interface simples (opcional)
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Parent = game:GetService("CoreGui")

local Frame = Instance.new("Frame")
Frame.Size = UDim2.new(0, 200, 0, 100)
Frame.Position = UDim2.new(0.5, -100, 0.5, -50)
Frame.BackgroundTransparency = 0.5
Frame.BackgroundColor3 = Color3.new(0, 0, 0)
Frame.Parent = ScreenGui

local Title = Instance.new("TextLabel")
Title.Text = "AUT Server Hop"
Title.Size = UDim2.new(1, 0, 0, 30)
Title.BackgroundTransparency = 1
Title.TextColor3 = Color3.new(1, 1, 1)
Title.Parent = Frame

local HopButton = Instance.new("TextButton")
HopButton.Text = "Hop Servidor"
HopButton.Size = UDim2.new(0.8, 0, 0, 30)
HopButton.Position = UDim2.new(0.1, 0, 0.5, 0)
HopButton.Parent = Frame

HopButton.MouseButton1Click:Connect(serverHop)

-- Auto-hop opcional (descomente se quiser hop automático)
-- while true do
--     local players = #Players:GetPlayers()
--     if players < MIN_PLAYERS or players > MAX_PLAYERS then
--         serverHop()
--     end
--     wait(60) -- Verificar a cada minuto
-- end

print("Script de Server Hop para A Universal Time carregado!")
print("Clique no botão 'Hop Servidor' para trocar de servidor.")

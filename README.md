-- Servidor Hop para A Universal Time (Roblox)
-- ID do Jogo: 5130598377
-- Versão Corrigida

local Players = game:GetService("Players")
local TeleportService = game:GetService("TeleportService")
local HttpService = game:GetService("HttpService")
local LocalPlayer = Players.LocalPlayer

-- Verifica se o jogo é A Universal Time
if game.PlaceId ~= 5130598377 then
    warn("Este script é apenas para A Universal Time (ID: 5130598377)")
    return
end

-- Tenta carregar a Rayfield
local Rayfield = nil
local success, errorMsg = pcall(function()
    Rayfield = loadstring(game:HttpGet('https://raw.githubusercontent.com/shlexware/Rayfield/main/source'))()
end)

if not success or not Rayfield then
    warn("Falha ao carregar Rayfield: " .. tostring(errorMsg))
    -- Continua sem interface, apenas com funcionalidades básicas
end

-- Configurações
local settings = {
    MIN_PLAYERS = 5,
    MAX_PLAYERS = 15,
    MAX_PING = 200,
    DELAY = 30,
    AUTO_HOP = true,
    PREFER_PRIVATE = false
}

-- Função para criar a interface (se Rayfield estiver disponível)
local function createUI()
    if not Rayfield then return end
    
    local Window = Rayfield:CreateWindow({
        Name = "AUT Server Hop (Corrigido)",
        LoadingTitle = "A Universal Time Server Hopper",
        LoadingSubtitle = "Versão Corrigida",
        ConfigurationSaving = {
            Enabled = true,
            FolderName = "AUT_Hopper_Fixed",
            FileName = "Config"
        },
        Discord = {
            Enabled = false,
            Invite = "noinvitelink",
            RememberJoins = true
        },
        KeySystem = false
    })

    local MainTab = Window:CreateTab("Principal", 4483362458)
    local SettingsTab = Window:CreateTab("Configurações", 4483362458)
    
    -- Elementos da UI
    MainTab:CreateSection("Status do Servidor")
    local playerCountLabel = MainTab:CreateLabel("Jogadores: " .. #Players:GetPlayers())
    local pingLabel = MainTab:CreateLabel("Ping: Calculando...")
    local serverIdLabel = MainTab:CreateLabel("Server ID: " .. game.JobId)
    
    MainTab:CreateSection("Controles")
    MainTab:CreateButton({
        Name = "Trocar de Servidor Agora",
        Callback = function()
            hopServer()
        end,
    })

    MainTab:CreateToggle({
        Name = "Auto Hop",
        CurrentValue = settings.AUTO_HOP,
        Flag = "AutoHopToggle",
        Callback = function(value)
            settings.AUTO_HOP = value
        end,
    })

    -- Configurações
    SettingsTab:CreateSection("Preferências")
    SettingsTab:CreateSlider({
        Name = "Mínimo de Jogadores",
        Range = {1, 30},
        Increment = 1,
        Suffix = "jogadores",
        CurrentValue = settings.MIN_PLAYERS,
        Flag = "MinPlayersSlider",
        Callback = function(value)
            settings.MIN_PLAYERS = value
        end,
    })

    SettingsTab:CreateSlider({
        Name = "Máximo de Jogadores",
        Range = {1, 30},
        Increment = 1,
        Suffix = "jogadores",
        CurrentValue = settings.MAX_PLAYERS,
        Flag = "MaxPlayersSlider",
        Callback = function(value)
            settings.MAX_PLAYERS = value
        end,
    })

    -- Atualiza a UI periodicamente
    coroutine.wrap(function()
        while true do
            playerCountLabel:Set("Jogadores: " .. #Players:GetPlayers())
            pingLabel:Set("Ping: " .. math.floor(getPing()) .. "ms")
            wait(2)
        end
    end)()
end

-- Função para obter ping
local function getPing()
    local success, ping = pcall(function()
        return game:GetService("Stats").Network.ServerStatsItem["Data Ping"]:GetValue()
    end)
    return success and ping or 999
end

-- Função para obter lista de servidores com tratamento de erros
local function getServerList()
    local success, result = pcall(function()
        local url = "https://games.roblox.com/v1/games/" .. game.PlaceId .. "/servers/Public?sortOrder=Asc&limit=100"
        return HttpService:JSONDecode(game:HttpGet(url))
    end)
    
    if success and result and result.data then
        return result.data
    else
        warn("Erro ao obter lista de servidores: " .. tostring(result))
        return {}
    end
end

-- Função para encontrar o melhor servidor
local function findBestServer()
    local servers = getServerList()
    if #servers == 0 then return nil end
    
    local bestServer = nil
    local currentPlayers = #Players:GetPlayers()
    
    for _, server in ipairs(servers) do
        if server.id ~= game.JobId then
            local playerCount = server.playing or 0
            local maxPlayers = server.maxPlayers or 30
            
            if playerCount >= settings.MIN_PLAYERS and 
               playerCount <= settings.MAX_PLAYERS and 
               playerCount < maxPlayers then
                
                if not bestServer or playerCount < (bestServer.playing or math.huge) then
                    bestServer = server
                end
            end
        end
    end
    
    return bestServer
end

-- Função principal para trocar de servidor
local function hopServer()
    print("Iniciando procura por servidor melhor...")
    
    local bestServer = findBestServer()
    
    if bestServer then
        print("Servidor encontrado! Jogadores:", bestServer.playing)
        
        if Rayfield then
            Rayfield:Notify({
                Title = "Trocando de Servidor",
                Content = "Encontrado servidor com " .. bestServer.playing .. " jogadores",
                Duration = 3,
                Image = 4483362458,
            })
        end
        
        TeleportService:TeleportToPlaceInstance(game.PlaceId, bestServer.id, LocalPlayer)
    else
        print("Nenhum servidor adequado encontrado.")
        
        if Rayfield then
            Rayfield:Notify({
                Title = "Nenhum Servidor Encontrado",
                Content = "Tente ajustar as configurações",
                Duration = 3,
                Image = 4483362458,
            })
        end
    end
end

-- Verifica se deve trocar de servidor
local function shouldHop()
    local playerCount = #Players:GetPlayers()
    local ping = getPing()
    
    return playerCount > settings.MAX_PLAYERS or ping > settings.MAX_PING
end

-- Inicialização
createUI()

-- Loop principal para auto hop
coroutine.wrap(function()
    while true do
        if settings.AUTO_HOP and shouldHop() then
            local success, err = pcall(hopServer)
            if not success then
                warn("Erro no auto hop: " .. tostring(err))
            end
        end
        wait(settings.DELAY)
    end
end)()

print("AUT Server Hop iniciado com sucesso!")

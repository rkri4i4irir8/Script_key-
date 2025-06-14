-- Servidor Hop para A Universal Time (Roblox)
-- ID do Jogo: 5130598377

local Players = game:GetService("Players")
local TeleportService = game:GetService("TeleportService")
local HttpService = game:GetService("HttpService")
local LocalPlayer = Players.LocalPlayer

-- Carrega a biblioteca Rayfield
local Rayfield = loadstring(game:HttpGet('https://raw.githubusercontent.com/shlexware/Rayfield/main/source'))()

-- Configurações padrão
local settings = {
    MIN_PLAYERS = 5,
    MAX_PLAYERS = 15,
    MAX_PING = 200,
    DELAY = 30,
    AUTO_HOP = true,
    PREFER_PRIVATE = false
}

-- Cria a janela principal
local Window = Rayfield:CreateWindow({
    Name = "AUT Server Hop",
    LoadingTitle = "A Universal Time Server Hopper",
    LoadingSubtitle = "by github.com/seu-usuario",
    ConfigurationSaving = {
        Enabled = true,
        FolderName = "AUT_Hopper",
        FileName = "Config"
    },
    Discord = {
        Enabled = false,
        Invite = "noinvitelink",
        RememberJoins = true
    },
    KeySystem = false
})

-- Tab principal
local MainTab = Window:CreateTab("Principal", 4483362458)
local SettingsTab = Window:CreateTab("Configurações", 4483362458)
local InfoTab = Window:CreateTab("Informações", 4483362458)

-- Elementos da UI
local playerCountLabel
local pingLabel
local serverIdLabel
local hopButton
local autoHopToggle

-- Função para atualizar as informações na UI
local function updateUI()
    playerCountLabel:Set("Jogadores: " .. #Players:GetPlayers())
    pingLabel:Set("Ping: " .. math.floor(getPing()) .. "ms")
    serverIdLabel:Set("Server ID: " .. game.JobId)
end

-- Função para obter lista de servidores
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

-- Função para medir ping
local function getPing()
    local stats = game:GetService("Stats")
    local ping = stats.Network.ServerStatsItem["Data Ping"]:GetValue()
    return ping
end

-- Função para verificar se deve trocar de servidor
local function shouldHop()
    local playerCount = #Players:GetPlayers()
    local ping = getPing()
    
    if playerCount > settings.MAX_PLAYERS or ping > settings.MAX_PING then
        return true
    end
    
    return false
end

-- Função para encontrar o melhor servidor
local function findBestServer()
    local servers = getServerList()
    local bestServer = nil
    local currentPlayers = #Players:GetPlayers()
    
    for _, server in ipairs(servers) do
        if server.id ~= game.JobId then
            local playerCount = server.playing or 0
            local maxPlayers = server.maxPlayers or 30
            
            -- Verifica preferências
            local isPrivate = server.vip or false
            local meetsConditions = true
            
            if settings.PREFER_PRIVATE and not isPrivate then
                meetsConditions = false
            end
            
            if playerCount >= settings.MIN_PLAYERS and playerCount <= settings.MAX_PLAYERS and playerCount < maxPlayers and meetsConditions then
                if not bestServer then
                    bestServer = server
                else
                    -- Prioriza servidores com menos jogadores e que sejam privados (se PREFER_PRIVATE estiver ativo)
                    if settings.PREFER_PRIVATE then
                        if isPrivate and (not bestServer.vip or playerCount < (bestServer.playing or math.huge)) then
                            bestServer = server
                        end
                    elseif playerCount < (bestServer.playing or math.huge) then
                        bestServer = server
                    end
                end
            end
        end
    end
    
    return bestServer
end

-- Função principal para trocar de servidor
local function hopServer()
    if not shouldHop() and settings.AUTO_HOP then
        print("Servidor atual ainda está bom. Não há necessidade de trocar.")
        return
    end
    
    local bestServer = findBestServer()
    
    if bestServer then
        print("Trocando para servidor com " .. bestServer.playing .. " jogadores...")
        Rayfield:Notify({
            Title = "Trocando de Servidor",
            Content = "Conectando a um servidor com " .. bestServer.playing .. " jogadores...",
            Duration = 3,
            Image = 4483362458,
            Actions = {
                Ignore = {
                    Name = "Ok",
                    Callback = function()
                    end
                },
            },
        })
        TeleportService:TeleportToPlaceInstance(game.PlaceId, bestServer.id, LocalPlayer)
    else
        print("Não foi encontrado um servidor melhor no momento.")
        Rayfield:Notify({
            Title = "Nenhum Servidor Encontrado",
            Content = "Não foi encontrado um servidor que atenda aos critérios atuais.",
            Duration = 3,
            Image = 4483362458,
            Actions = {
                Ignore = {
                    Name = "Ok",
                    Callback = function()
                    end
                },
            },
        })
    end
end

-- Cria os elementos da UI
MainTab:CreateSection("Status do Servidor")
playerCountLabel = MainTab:CreateLabel("Jogadores: " .. #Players:GetPlayers())
pingLabel = MainTab:CreateLabel("Ping: " .. math.floor(getPing()) .. "ms")
serverIdLabel = MainTab:CreateLabel("Server ID: " .. game.JobId)

MainTab:CreateSection("Controles")
hopButton = MainTab:CreateButton({
    Name = "Trocar de Servidor Agora",
    Callback = function()
        hopServer()
    end,
})

autoHopToggle = MainTab:CreateToggle({
    Name = "Auto Hop",
    CurrentValue = settings.AUTO_HOP,
    Flag = "AutoHopToggle",
    Callback = function(value)
        settings.AUTO_HOP = value
    end,
})

-- Configurações
SettingsTab:CreateSection("Preferências de Servidor")
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

SettingsTab:CreateSlider({
    Name = "Ping Máximo",
    Range = {50, 500},
    Increment = 10,
    Suffix = "ms",
    CurrentValue = settings.MAX_PING,
    Flag = "MaxPingSlider",
    Callback = function(value)
        settings.MAX_PING = value
    end,
})

SettingsTab:CreateSlider({
    Name = "Intervalo de Verificação",
    Range = {10, 120},
    Increment = 5,
    Suffix = "segundos",
    CurrentValue = settings.DELAY,
    Flag = "DelaySlider",
    Callback = function(value)
        settings.DELAY = value
    end,
})

SettingsTab:CreateToggle({
    Name = "Preferir Servidores Privados",
    CurrentValue = settings.PREFER_PRIVATE,
    Flag = "PreferPrivateToggle",
    Callback = function(value)
        settings.PREFER_PRIVATE = value
    end,
})

-- Informações
InfoTab:CreateSection("Sobre")
InfoTab:CreateLabel("AUT Server Hop v1.0")
InfoTab:CreateLabel("Desenvolvido para A Universal Time")
InfoTab:CreateLabel("ID do Jogo: 5130598377")

InfoTab:CreateSection("Instruções")
InfoTab:CreateLabel("- Auto Hop troca automaticamente quando necessário")
InfoTab:CreateLabel("- Ajuste as configurações para suas preferências")
InfoTab:CreateLabel("- Clique no botão para trocar manualmente")

-- Atualiza a UI periodicamente
coroutine.wrap(function()
    while true do
        updateUI()
        wait(1)
    end
end)()

-- Loop principal para auto hop
coroutine.wrap(function()
    while true do
        if settings.AUTO_HOP then
            local success, err = pcall(hopServer)
            if not success then
                warn("Erro durante o servidor hop: " .. tostring(err))
            end
        end
        wait(settings.DELAY)
    end
end)()

-- Notificação de inicialização
Rayfield:Notify({
    Title = "AUT Server Hop Iniciado",
    Content = "O servidor hop foi iniciado com sucesso!",
    Duration = 5,
    Image = 4483362458,
    Actions = {
        Ignore = {
            Name = "Ok",
            Callback = function()
            end
        },
    },
})

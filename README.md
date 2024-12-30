-- Pegando os serviços necessários
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")

-- Tabela para armazenar as linhas criadas
local lines = {}

-- Função para criar ou atualizar a linha entre dois jogadores
local function updateLineBetweenPlayers(player1, player2)
    -- Pegando as posições dos jogadores
    local char1 = player1.Character
    local char2 = player2.Character

    -- Garantindo que ambos os jogadores tenham personagens
    if char1 and char2 then
        local position1 = char1.HumanoidRootPart.Position
        local position2 = char2.HumanoidRootPart.Position

        -- Verificando se já existe uma linha para esses dois jogadores
        local lineKey = player1.UserId .. "-" .. player2.UserId
        if not lines[lineKey] then
            -- Criando a parte visual da linha (usando uma "Part" no Roblox)
            local line = Instance.new("Part")
            line.Anchored = true
            line.CanCollide = false
            line.Size = Vector3.new(0.2, 0.2, (position1 - position2).Magnitude)
            line.Color = Color3.fromRGB(255, 0, 0) -- Cor vermelha
            line.Position = (position1 + position2) / 2 -- Posição no meio dos dois pontos
            line.CFrame = CFrame.new(position1, position2) -- Rotaciona a parte para conectar os dois pontos

            -- Criando o BillboardGui para mostrar o nome do jogador
            local billboard = Instance.new("BillboardGui")
            billboard.Parent = line
            billboard.Adornee = line
            billboard.Size = UDim2.new(0, 100, 0, 50) -- Tamanho da área do nome
            billboard.StudsOffset = Vector3.new(0, 2, 0) -- Distância do nome em relação à linha

            -- Criando o TextLabel para mostrar o nome do jogador
            local textLabel = Instance.new("TextLabel")
            textLabel.Parent = billboard
            textLabel.Text = player2.Name -- Nome do jogador
            textLabel.TextColor3 = Color3.fromRGB(255, 255, 255) -- Cor do texto
            textLabel.BackgroundTransparency = 1 -- Remover fundo
            textLabel.TextSize = 14 -- Tamanho do texto
            textLabel.TextStrokeTransparency = 0.8 -- Transparência da borda do texto

            -- Adicionando a linha ao jogo e armazenando ela
            line.Parent = game.Workspace
            lines[lineKey] = line
        else
            -- Atualizando a linha existente
            local line = lines[lineKey]
            line.Size = Vector3.new(0.2, 0.2, (position1 - position2).Magnitude)
            line.Position = (position1 + position2) / 2
            line.CFrame = CFrame.new(position1, position2)

            -- Atualizando o BillboardGui
            local billboard = line:FindFirstChildOfClass("BillboardGui")
            if billboard then
                local textLabel = billboard:FindFirstChildOfClass("TextLabel")
                if textLabel then
                    textLabel.Text = player2.Name -- Atualiza o nome
                end
            end
        end
    end
end

-- Função para garantir que as linhas sigam os jogadores em movimento
local function connectPlayers()
    local localPlayer = Players.LocalPlayer

    -- Garantindo que o personagem do jogador local esteja carregado
    if localPlayer.Character then
        -- Atualizando as linhas a cada quadro (Heartbeat)
        RunService.Heartbeat:Connect(function()
            -- Criar ou atualizar a linha entre o jogador local e os outros jogadores
            for _, player in pairs(Players:GetPlayers()) do
                if player ~= localPlayer and player.Character then
                    updateLineBetweenPlayers(localPlayer, player)
                end
            end
        end)
    end
end

-- Função para remover a linha de um jogador que saiu do jogo
local function onPlayerRemoving(player)
    -- Remover a linha associada a esse jogador
    for _, otherPlayer in pairs(Players:GetPlayers()) do
        if otherPlayer ~= player then
            local lineKey = player.UserId .. "-" .. otherPlayer.UserId
            if lines[lineKey] then
                lines[lineKey]:Destroy()  -- Apagar a linha
                lines[lineKey] = nil      -- Remover da tabela
            end
        end
    end
end

-- Conectar o evento para quando um jogador sai do jogo
Players.PlayerRemoving:Connect(onPlayerRemoving)

-- Chama a função para conectar os jogadores
connectPlayers()

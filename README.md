-- Script para copiar dados do jogo automaticamente
-- Compatível com Roblox Studio/Executors

local HttpService = game:GetService("HttpService")
local Players = game:GetService("Players")

-- Função para serializar tabelas
local function serializeTable(tbl, indent)
    indent = indent or 0
    local str = "{\n"
    local spacing = string.rep("  ", indent + 1)
    
    for k, v in pairs(tbl) do
        str = str .. spacing
        
        if type(k) == "string" then
            str = str .. '["' .. k .. '"] = '
        else
            str = str .. "[" .. tostring(k) .. "] = "
        end
        
        if type(v) == "table" then
            str = str .. serializeTable(v, indent + 1)
        elseif type(v) == "string" then
            str = str .. '"' .. v .. '"'
        else
            str = str .. tostring(v)
        end
        
        str = str .. ",\n"
    end
    
    str = str .. string.rep("  ", indent) .. "}"
    return str
end

-- Função para copiar propriedades de um objeto
local function copyProperties(obj)
    local data = {
        Name = obj.Name,
        ClassName = obj.ClassName,
        Parent = obj.Parent and obj.Parent.Name or "nil"
    }
    
    -- Tentar copiar propriedades comuns
    local success, result = pcall(function()
        if obj:IsA("BasePart") then
            data.Position = tostring(obj.Position)
            data.Size = tostring(obj.Size)
            data.Color = tostring(obj.Color)
            data.Material = tostring(obj.Material)
            data.Transparency = obj.Transparency
        elseif obj:IsA("Model") then
            data.PrimaryPart = obj.PrimaryPart and obj.PrimaryPart.Name or "nil"
        end
    end)
    
    return data
end

-- Função para copiar hierarquia de objetos
local function copyHierarchy(parent, maxDepth)
    maxDepth = maxDepth or 5
    if maxDepth <= 0 then return {} end
    
    local hierarchy = {}
    
    for _, child in ipairs(parent:GetChildren()) do
        local childData = copyProperties(child)
        childData.Children = copyHierarchy(child, maxDepth - 1)
        table.insert(hierarchy, childData)
    end
    
    return hierarchy
end

-- Função principal para copiar todos os dados
local function copyGameData()
    print("=== INICIANDO CÓPIA DE DADOS DO JOGO ===")
    
    local gameData = {
        GameInfo = {
            Name = game.Name,
            PlaceId = game.PlaceId,
            JobId = game.JobId,
            CreatorType = tostring(game.CreatorType),
            CreatorId = game.CreatorId
        },
        
        Workspace = {},
        ReplicatedStorage = {},
        ServerStorage = {},
        Lighting = {},
        Players = {}
    }
    
    -- Copiar Workspace
    print("Copiando Workspace...")
    gameData.Workspace = copyHierarchy(game.Workspace, 3)
    
    -- Copiar ReplicatedStorage
    print("Copiando ReplicatedStorage...")
    if game:FindFirstChild("ReplicatedStorage") then
        gameData.ReplicatedStorage = copyHierarchy(game.ReplicatedStorage, 4)
    end
    
    -- Copiar Lighting
    print("Copiando Lighting...")
    gameData.Lighting = {
        Brightness = game.Lighting.Brightness,
        ClockTime = game.Lighting.ClockTime,
        FogEnd = game.Lighting.FogEnd,
        FogStart = game.Lighting.FogStart,
        Ambient = tostring(game.Lighting.Ambient),
        OutdoorAmbient = tostring(game.Lighting.OutdoorAmbient)
    }
    
    -- Copiar informações dos jogadores
    print("Copiando dados dos jogadores...")
    for _, player in ipairs(Players:GetPlayers()) do
        table.insert(gameData.Players, {
            Name = player.Name,
            UserId = player.UserId,
            AccountAge = player.AccountAge,
            Team = player.Team and player.Team.Name or "None"
        })
    end
    
    -- Converter para string
    local jsonData = serializeTable(gameData)
    
    -- Copiar para clipboard (se suportado)
    if setclipboard then
        setclipboard(jsonData)
        print("✓ Dados copiados para a área de transferência!")
    end
    
    -- Salvar em arquivo (se suportado)
    if writefile then
        local fileName = "GameData_" .. game.PlaceId .. "_" .. os.time() .. ".txt"
        writefile(fileName, jsonData)
        print("✓ Dados salvos em: " .. fileName)
    end
    
    print("=== CÓPIA CONCLUÍDA ===")
    print("Total de caracteres: " .. #jsonData)
    
    return gameData
end

-- Executar automaticamente ao entrar no jogo
wait(2) -- Aguardar carregamento completo
copyGameData()

-- Criar comando para copiar novamente
_G.CopyGameData = copyGameData

print([[
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  Script de Cópia de Dados Carregado!
  
  Para copiar novamente, use:
  _G.CopyGameData()
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
]])

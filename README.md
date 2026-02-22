-- Script com loop: Yuno -> Wood -> Asta -> Yuno (repetindo)
-- Coloque este script em StarterPlayer > StarterPlayerScripts

local Players = game:GetService("Players")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local Workspace = game:GetService("Workspace")

local player = Players.LocalPlayer
local character = player.Character or player.CharacterAdded:Wait()
local humanoidRootPart = character:WaitForChild("HumanoidRootPart")

-- Configurações
local NPC_YUNO = "Yuno"
local NPC_ASTA = "Asta"
local QUEST_TYPE = "questpls"
local QUEST_EXTRA = "DeliverGreenJuice"
local REMOTE_NAME = "MainRemote"
local TREE_INDEX = 54
local WOOD_INDEX = 2

-- Variáveis de controle
local isLooping = true
local loopCount = 0
local MAX_LOOPS = 0 -- 0 = loop infinito

-- Função para teleportar para um NPC
local function teleportToNPC(npcName)
    local npc = Workspace.NPCs:FindFirstChild(npcName)
    
    if npc then
        local npcRootPart = npc:FindFirstChild("HumanoidRootPart")
        
        if npcRootPart then
            humanoidRootPart.CFrame = npcRootPart.CFrame + Vector3.new(5, 0, 0)
            print("✓ Teleportado para: " .. npcName)
            return true
        else
            warn("✗ HumanoidRootPart não encontrado em " .. npcName)
            return false
        end
    else
        warn("✗ NPC " .. npcName .. " não encontrado")
        return false
    end
end

-- Função para teleportar para a pasta GIVEMEJUICE do Asta
local function teleportToAstaGiveJuice()
    local asta = Workspace.NPCs:FindFirstChild(NPC_ASTA)
    
    if asta then
        local giveJuice = asta:FindFirstChild("GIVEMEJUICE")
        
        if giveJuice then
            local giveJuicePart = giveJuice:IsA("BasePart") and giveJuice or giveJuice:FindFirstChildWhichIsA("BasePart")
            
            if giveJuicePart then
                humanoidRootPart.CFrame = giveJuicePart.CFrame + Vector3.new(0, 3, 0)
                print("✓ Teleportado para: Asta.GIVEMEJUICE")
                return true
            else
                warn("✗ BasePart não encontrado em Asta.GIVEMEJUICE")
                return false
            end
        else
            warn("✗ GIVEMEJUICE não encontrado em Asta")
            return false
        end
    else
        warn("✗ NPC Asta não encontrado")
        return false
    end
end

-- Função para aceitar a missão do Yuno
local function acceptQuestFromYuno()
    local mainRemote = ReplicatedStorage:WaitForChild(REMOTE_NAME)
    
    local args = {
        "pcgamer",
        {
            Extra = QUEST_EXTRA,
            Type = QUEST_TYPE,
            NpcName = NPC_YUNO
        }
    }
    
    mainRemote:FireServer(unpack(args))
    print("✓ Missão aceita: " .. QUEST_EXTRA .. " (Yuno)")
end

-- Função para entregar a missão no Asta
local function deliverQuestToAsta()
    local mainRemote = ReplicatedStorage:WaitForChild(REMOTE_NAME)
    
    local args = {
        "pcgamer",
        {
            Extra = QUEST_EXTRA,
            Type = QUEST_TYPE,
            NpcName = NPC_ASTA
        }
    }
    
    mainRemote:FireServer(unpack(args))
    print("✓ Missão entregue em: " .. NPC_ASTA)
end

-- Função para teleportar para a madeira
local function teleportToWood()
    local mapDetails = Workspace:FindFirstChild("THEMAP")
    
    if mapDetails then
        mapDetails = mapDetails:FindFirstChild("MapDetails")
        
        if mapDetails then
            local trees = mapDetails:FindFirstChild("Trees")
            
            if trees then
                local treeChildren = trees:GetChildren()
                
                if treeChildren[TREE_INDEX] then
                    local tree = treeChildren[TREE_INDEX]
                    local woodFolder = tree:FindFirstChild("Wood")
                    
                    if woodFolder then
                        local woodChildren = woodFolder:GetChildren()
                        
                        if woodChildren[WOOD_INDEX] then
                            local wood = woodChildren[WOOD_INDEX]
                            local woodPart = wood:IsA("BasePart") and wood or wood:FindFirstChildWhichIsA("BasePart")
                            
                            if woodPart then
                                humanoidRootPart.CFrame = woodPart.CFrame + Vector3.new(0, 3, 0)
                                print("✓ Teleportado para: Wood #" .. TREE_INDEX)
                                return true
                            end
                        end
                    end
                end
            end
        end
    end
    
    warn("✗ Falha ao teleportar para Wood")
    return false
end

-- Função principal do loop
local function runQuestLoop()
    while isLooping do
        loopCount = loopCount + 1
        print("\n" .. string.rep("=", 50))
        print(">>> CICLO #" .. loopCount .. " <<<")
        print(string.rep("=", 50))
        
        -- ETAPA 1: Teleportar para Yuno
        if not teleportToNPC(NPC_YUNO) then
            warn("Abortando ciclo...")
            break
        end
        wait(0.3)
        
        -- ETAPA 2: Aceitar missão do Yuno
        acceptQuestFromYuno()
        wait(0.3)
        
        -- ETAPA 3: Teleportar para a madeira
        if not teleportToWood() then
            warn("Abortando ciclo...")
            break
        end
        wait(0.3)
        
        -- ETAPA 4: Teleportar para Asta.GIVEMEJUICE
        if not teleportToAstaGiveJuice() then
            warn("Abortando ciclo...")
            break
        end
        wait(0.3)
        
        -- ETAPA 5: Entregar missão no Asta
        deliverQuestToAsta()
        wait(0.3)
        
        print(">>> CICLO #" .. loopCount .. " COMPLETO <<<\n")
        
        -- Verificar se deve continuar o loop
        if MAX_LOOPS > 0 and loopCount >= MAX_LOOPS then
            print("=== LIMITE DE CICLOS ATINGIDO ===")
            isLooping = false
        end
    end
    
    print("\n" .. string.rep("=", 50))
    print("=== LOOP FINALIZADO ===")
    print("Total de ciclos: " .. loopCount)
    print(string.rep("=", 50))
end

-- Iniciar o loop
print("=== INICIANDO LOOP DE QUESTS ===")
print("Pressione CTRL+C para parar o script")
runQuestLoop()

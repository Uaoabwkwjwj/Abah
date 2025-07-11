local ReplicatedStorage = game:GetService("ReplicatedStorage")
local Players = game:GetService("Players")
local StarterGui = game:GetService("StarterGui")

local ADMINS = {
    "nolyhaha",
    "sr_greg81",
    "djekejsn",
}

local remoteEvent = ReplicatedStorage:FindFirstChild("AdminCommandsRemote")
if not remoteEvent then
    remoteEvent = Instance.new("RemoteEvent")
    remoteEvent.Name = "AdminCommandsRemote"
    remoteEvent.Parent = ReplicatedStorage
end

local function isAdmin(player)
    for _, adminName in pairs(ADMINS) do
        if player.Name == adminName then
            return true
        end
    end
    return false
end

local function findPlayer(name)
    name = name:lower()
    for _, player in pairs(Players:GetPlayers()) do
        if player.Name:lower():find(name) then
            return player
        end
    end
    return nil
end

local function freezePlayer(targetPlayer)
    if targetPlayer and targetPlayer.Character then
        local humanoid = targetPlayer.Character:FindFirstChild("Humanoid")
        local rootPart = targetPlayer.Character:FindFirstChild("HumanoidRootPart")
        
        if humanoid and rootPart then
            humanoid.PlatformStand = true
            rootPart.Anchored = true
            
            local freezeEffect = Instance.new("SelectionBox")
            freezeEffect.Name = "FreezeEffect"
            freezeEffect.Adornee = targetPlayer.Character
            freezeEffect.Color3 = Color3.fromRGB(0, 162, 255)
            freezeEffect.LineThickness = 0.2
            freezeEffect.Transparency = 0.5
            freezeEffect.Parent = targetPlayer.Character
            
            return true
        end
    end
    return false
end

local function unfreezePlayer(targetPlayer)
    if targetPlayer and targetPlayer.Character then
        local humanoid = targetPlayer.Character:FindFirstChild("Humanoid")
        local rootPart = targetPlayer.Character:FindFirstChild("HumanoidRootPart")
        local freezeEffect = targetPlayer.Character:FindFirstChild("FreezeEffect")
        
        if humanoid and rootPart then
            humanoid.PlatformStand = false
            rootPart.Anchored = false
            
            if freezeEffect then
                freezeEffect:Destroy()
            end
            
            return true
        end
    end
    return false
end

local function killPlayer(targetPlayer)
    if targetPlayer and targetPlayer.Character then
        local humanoid = targetPlayer.Character:FindFirstChild("Humanoid")
        if humanoid then
            humanoid.Health = 0
            return true
        end
    end
    return false
end

local function sendMessageToAdmin(admin, message, isError)
    local color = isError and Color3.fromRGB(255, 100, 100) or Color3.fromRGB(100, 255, 100)
    
    local success, error = pcall(function()
        StarterGui:SetCore("ChatMakeSystemMessage", {
            Text = "[ADMIN] " .. message,
            Color = color,
            Font = Enum.Font.SourceSansBold,
            FontSize = Enum.FontSize.Size18
        })
    end)
end

local function processCommand(admin, command, targetName)
    if not isAdmin(admin) then
        sendMessageToAdmin(admin, "Você não tem permissão para usar comandos de admin!", true)
        return
    end
    
    if command == "freeze" then
        if not targetName or targetName == "" then
            sendMessageToAdmin(admin, "Uso: ;freeze [nome do jogador]", true)
            return
        end
        
        local targetPlayer = findPlayer(targetName)
        if not targetPlayer then
            sendMessageToAdmin(admin, "Jogador '" .. targetName .. "' não encontrado!", true)
            return
        end
        
        if freezePlayer(targetPlayer) then
            sendMessageToAdmin(admin, "Jogador " .. targetPlayer.Name .. " foi congelado!")
        else
            sendMessageToAdmin(admin, "Erro ao congelar " .. targetPlayer.Name, true)
        end
        
    elseif command == "kill" then
        if not targetName or targetName == "" then
            sendMessageToAdmin(admin, "Uso: ;kill [nome do jogador]", true)
            return
        end
        
        local targetPlayer = findPlayer(targetName)
        if not targetPlayer then
            sendMessageToAdmin(admin, "Jogador '" .. targetName .. "' não encontrado!", true)
            return
        end
        
        if killPlayer(targetPlayer) then
            sendMessageToAdmin(admin, "Jogador " .. targetPlayer.Name .. " foi eliminado!")
        else
            sendMessageToAdmin(admin, "Erro ao eliminar " .. targetPlayer.Name, true)
        end
        
    elseif command == "unfreeze" then
        if not targetName or targetName == "" then
            sendMessageToAdmin(admin, "Uso: ;unfreeze [nome do jogador]", true)
            return
        end
        
        local targetPlayer = findPlayer(targetName)
        if not targetPlayer then
            sendMessageToAdmin(admin, "Jogador '" .. targetName .. "' não encontrado!", true)
            return
        end
        
        if unfreezePlayer(targetPlayer) then
            sendMessageToAdmin(admin, "Jogador " .. targetPlayer.Name .. " foi descongelado!")
        else
            sendMessageToAdmin(admin, "Erro ao descongelar " .. targetPlayer.Name, true)
        end
        
    elseif command == "help" then
        sendMessageToAdmin(admin, "Comandos disponíveis:")
        sendMessageToAdmin(admin, ";freeze [jogador] - Congela o jogador")
        sendMessageToAdmin(admin, ";unfreeze [jogador] - Descongela o jogador")
        sendMessageToAdmin(admin, ";kill [jogador] - Elimina o jogador")
        sendMessageToAdmin(admin, ";admins - Lista os administradores")
        
    elseif command == "admins" then
        local adminList = "Administradores: " .. table.concat(ADMINS, ", ")
        sendMessageToAdmin(admin, adminList)
        
    else
        sendMessageToAdmin(admin, "Comando desconhecido: ;" .. command .. ". Use ;help para ver os comandos.", true)
    end
end

remoteEvent.OnServerEvent:Connect(function(player, command, targetName)
    processCommand(player, command, targetName)
end)

Players.PlayerAdded:Connect(function(player)
    if isAdmin(player) then
        wait(2)
        sendMessageToAdmin(player, "Bem-vindo, Admin! Use ;help para ver os comandos.")
    end
end)

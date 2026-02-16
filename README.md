local Rayfield = loadstring(game:HttpGet('https://sirius.menu/rayfield'))()
local VIM = game:GetService("VirtualInputManager")
local HttpService = game:GetService("HttpService")
local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer

-- Cria a janela principal
local Window = Rayfield:CreateWindow({
    Name = "Auto Farm Brainrots",
    LoadingTitle = "Carregando Farm...",
    LoadingSubtitle = "Integração por YeagerBad",
    ConfigurationSaving = {
        Enabled = true,
        FolderName = "BrainrotFarm",
        FileName = "Configuracoes"
    },
    Discord = {
        Enabled = false,
        Invite = "", 
        RememberJoins = true
    },
    KeySystem = false, 
})

-- ===========================
-- VARIÁVEIS GLOBAIS
-- ===========================
local BaseCFrame = CFrame.new(1, 5, 33)
local WebhookURL = "https://discord.com/api/webhooks/1468282617739739349/xZV3W-jvbg377UA9shLJASKG_NIZSNdx84dpfnYEf66LIHCtEgmQWRvYgn2-HcUop61I"

local Toggles = {
    Celestial = true,
    Godly = true,
    Rare = false
}

local AutoFarmEnabled = false
local IsFarming = false
local WanderRadius = 15 -- Raio máximo que ele pode se afastar do centro

-- Teclas de Movimentação
local MoveKeys = {Enum.KeyCode.W, Enum.KeyCode.A, Enum.KeyCode.S, Enum.KeyCode.D}

-- ===========================
-- FUNÇÕES UTILITÁRIAS
-- ===========================

-- Dispara o Webhook para o Discord
local function SendWebhook(displayName, mutation)
    local httprequest = (request or http_request or syn and syn.request)
    if not httprequest then return end

    local titleText = string.format("🧠 [%s] [%s] capturado!", tostring(displayName), tostring(mutation))

    local data = {
        ["embeds"] = {{
            ["title"] = titleText,
            ["color"] = 5814783,
            ["fields"] = {
                {["name"] = "Nome (DisplayName)", ["value"] = tostring(displayName), ["inline"] = true},
                {["name"] = "Mutação", ["value"] = tostring(mutation), ["inline"] = true}
            },
            ["footer"] = {
                ["text"] = "Auto Farm by YeagerBad"
            }
        }}
    }

    httprequest({
        Url = WebhookURL,
        Method = "POST",
        Headers = {["Content-Type"] = "application/json"},
        Body = HttpService:JSONEncode(data)
    })
end

-- Lógica principal de captura de um único Brainrot
local function FarmBrainrot(brainrot)
    if IsFarming then return end
    IsFarming = true

    local char = LocalPlayer.Character
    local root = char and char:FindFirstChild("HumanoidRootPart")
    local humanoid = char and char:FindFirstChild("Humanoid")
    if not root or not humanoid then IsFarming = false return end

    -- Salva os atributos antes de sumir
    local displayName = brainrot:GetAttribute("DisplayName") or brainrot.Name
    local mutation = brainrot:GetAttribute("Mutation") or "Nenhuma"

    -- Teleporta para o Brainrot
    root.CFrame = brainrot:GetPivot()
    task.wait(0.3)

    -- MÁGICA DO EXPLOIT: Procura o botão (ProximityPrompt) dentro do Brainrot
    local prompt = brainrot:FindFirstChildWhichIsA("ProximityPrompt", true)
    
    if prompt then
        if fireproximityprompt then
            fireproximityprompt(prompt)
        else
            prompt.HoldDuration = 0
            prompt:InputHoldBegin()
            task.wait(0.1)
            prompt:InputHoldEnd()
        end
    else
        VIM:SendKeyEvent(true, Enum.KeyCode.E, false, game)
    end
    
    -- Loop inteligente: Espera até 5 segundos para colocar na mão
    local elapsed = 0
    while elapsed < 5.0 do
        task.wait(0.1)
        elapsed = elapsed + 0.1
        if char:FindFirstChild(brainrot.Name) or LocalPlayer.Backpack:FindFirstChild(brainrot.Name) then
            break
        end
    end

    VIM:SendKeyEvent(false, Enum.KeyCode.E, false, game)
    task.wait(0.2)
    
    -- ==========================================
    -- LÓGICA DE ENTREGA NA BASE (WASD SIMULADO)
    -- ==========================================
    root.CFrame = BaseCFrame
    task.wait(0.1)
    
    -- Fica apertando teclas aleatórias para acionar o detector da Base
    local depositTimeout = 0
    while char:FindFirstChild(brainrot.Name) and depositTimeout < 4.0 do
        local randomKey = MoveKeys[math.random(1, #MoveKeys)]
        
        -- Aperta a tecla por 0.3 segundos
        VIM:SendKeyEvent(true, randomKey, false, game)
        task.wait(0.3)
        VIM:SendKeyEvent(false, randomKey, false, game)
        
        depositTimeout = depositTimeout + 0.3
    end

    -- Força soltar todas as teclas para evitar andar sozinho
    for _, key in ipairs(MoveKeys) do
        VIM:SendKeyEvent(false, key, false, game)
    end

    -- Envia o webhook agora que temos certeza que foi entregue
    SendWebhook(displayName, mutation)
    
    IsFarming = false
end

-- ===========================
-- LOOPS PRINCIPAIS
-- ===========================

-- Loop de Farm (Procura Brainrots)
task.spawn(function()
    while task.wait(0.5) do
        if AutoFarmEnabled and not IsFarming then
            local gameFolder = workspace:FindFirstChild("GameFolder")
            local brainrotsFolder = gameFolder and gameFolder:FindFirstChild("Brainrots")
            
            if brainrotsFolder then
                local foldersToScan = {}
                
                if Toggles.Celestial and brainrotsFolder:FindFirstChild("Celestial") then table.insert(foldersToScan, brainrotsFolder.Celestial) end
                if Toggles.Godly and brainrotsFolder:FindFirstChild("Godly") then table.insert(foldersToScan, brainrotsFolder.Godly) end
                if Toggles.Rare and brainrotsFolder:FindFirstChild("Rare") then table.insert(foldersToScan, brainrotsFolder.Rare) end
                
                for _, folder in ipairs(foldersToScan) do
                    local items = folder:GetChildren()
                    if #items > 0 then
                        for _, item in ipairs(items) do
                            FarmBrainrot(item)
                            break 
                        end
                        break 
                    end
                end
            end
        end
    end
end)

-- Loop de Movimentação na Base com Teclado (Anti-AFK em Círculo)
task.spawn(function()
    while task.wait(0.5) do 
        if AutoFarmEnabled and not IsFarming then
            local char = LocalPlayer.Character
            local root = char and char:FindFirstChild("HumanoidRootPart")
            
            if root then
                -- Checa a distância do centro
                local distanceFromCenter = (root.Position - BaseCFrame.Position).Magnitude
                
                if distanceFromCenter > WanderRadius then
                    -- Se passou do limite, teleporta de volta pro centro (cerca invisível)
                    root.CFrame = BaseCFrame
                    task.wait(0.2)
                else
                    -- Simula o toque no teclado por um tempo curto
                    local randomKey = MoveKeys[math.random(1, #MoveKeys)]
                    local duration = math.random(3, 8) / 10 -- Segura a tecla de 0.3s a 0.8s
                    
                    VIM:SendKeyEvent(true, randomKey, false, game)
                    task.wait(duration)
                    VIM:SendKeyEvent(false, randomKey, false, game)
                end
            end
        end
    end
end)

-- ===========================
-- INTERFACE (GUI)
-- ===========================
local MainTab = Window:CreateTab("Auto Farm", "bot")

MainTab:CreateToggle({
    Name = "Ligar Auto Farm",
    CurrentValue = false,
    Flag = "MasterToggle",
    Callback = function(Value)
        AutoFarmEnabled = Value
        if Value then
            Rayfield:Notify({Title = "Auto Farm", Content = "Farm Iniciado!", Duration = 2})
        else
            Rayfield:Notify({Title = "Auto Farm", Content = "Farm Parado!", Duration = 2})
        end
    end,
})

-- Slider para o Raio de Movimento
MainTab:CreateSlider({
    Name = "Limites da Base (Raio)",
    Range = {5, 100},
    Increment = 5,
    Suffix = "Studs",
    CurrentValue = 15,
    Flag = "WanderRadiusSlider",
    Callback = function(Value)
        WanderRadius = Value
    end,
})

MainTab:CreateSection("Raridades para Farmar")

MainTab:CreateToggle({
    Name = "Farmar Celestial",
    CurrentValue = true,
    Flag = "ToggleCelestial",
    Callback = function(Value)
        Toggles.Celestial = Value
    end,
})

MainTab:CreateToggle({
    Name = "Farmar Godly",
    CurrentValue = true,
    Flag = "ToggleGodly",
    Callback = function(Value)
        Toggles.Godly = Value
    end,
})

MainTab:CreateToggle({
    Name = "Farmar Rare (Para Testes)",
    CurrentValue = false,
    Flag = "ToggleRare",
    Callback = function(Value)
        Toggles.Rare = Value
    end,
})

Rayfield:Notify({
    Title = "Script Atualizado",
    Content = "Simulação WASD aplicada!",
    Duration = 3.0,
})

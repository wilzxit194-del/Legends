local Rayfield = loadstring(game:HttpGet('https://sirius.menu/rayfield'))()

local Players = game:GetService("Players")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local StatsService = game:GetService("Stats")
local VirtualUser = game:GetService("VirtualUser")
local VirtualInputManager = game:GetService("VirtualInputManager")

local player = Players.LocalPlayer

-- Remotes do Jogo
local rEvents = ReplicatedStorage:WaitForChild("rEvents", 10)
local openRemote = rEvents and rEvents:WaitForChild("openCrystalRemote", 10)
local autoEvolveRemote = rEvents and rEvents:WaitForChild("autoEvolveRemote", 10)
local sellPetEvent = rEvents and rEvents:WaitForChild("sellPetEvent", 10)
local rebirthRemote = rEvents and rEvents:WaitForChild("rebirthRemote", 10)

-- Estados de Farm
local states = {
    masterToggle = false,
    autoOpen = false,
    autoEvolve = false,
    autoSell = false,
    autoReps = false,
    autoRebirth = false,
    antiAFK = false,
    openAmount = 1
}

-- Métrica de Ganhos e Tempo
local metrics = {
    totalOpened = 0,
    totalSold = 0,
    totalRebirths = 0,
    afkTime = 0,
    sessionTime = 0
}

-- Função Auxiliar para Formatar Tempo (HH:MM:SS)
local function formatTime(seconds)
    local hours = math.floor(seconds / 3600)
    local mins = math.floor((seconds % 3600) / 60)
    local secs = seconds % 60
    return string.format("%02dh %02dm %02ds", hours, mins, secs)
end

-- Interface Gráfica Principal
local Window = Rayfield:CreateWindow({
    Name = "OWNER AFK PAID HUB | ULTRA SPEED",
    LoadingTitle = "⚡ OWNER AFK PAID HUB",
    LoadingSubtitle = "by Roblox Dev | Maximum Performance Edition",
    ConfigurationSaving = {
        Enabled = false
    },
    KeySystem = false,
    CustomImage = "rbxassetid://1228978229",
    Theme = "Dark"
})

-- Aplicar Imagem no Fundo da Interface
task.spawn(function()
    pcall(function()
        local coreGui = game:GetService("CoreGui")
        local rayfieldGui = coreGui:FindFirstChild("Rayfield")
        if rayfieldGui then
            local mainFrame = rayfieldGui:FindFirstChild("Main", true)
            if mainFrame and not mainFrame:FindFirstChild("CustomBackground") then
                local bg = Instance.new("ImageLabel")
                bg.Name = "CustomBackground"
                bg.Size = UDim2.new(1, 0, 1, 0)
                bg.Position = UDim2.new(0, 0, 0, 0)
                bg.Image = "rbxassetid://1228978229"
                bg.ImageTransparency = 0.85
                bg.BackgroundTransparency = 1
                bg.ZIndex = 0
                bg.Parent = mainFrame
            end
        end
    end)
end)

-- =======================================================
-- TAB 1: FARMING & AUTOMAÇÃO
-- =======================================================
local FarmTab = Window:CreateTab("🌾 Opções de Farm", 1228978229)

FarmTab:CreateToggle({
    Name = "⚡ Ativar Sistema Geral",
    CurrentValue = false,
    Callback = function(Value)
        states.masterToggle = Value
    end,
})

FarmTab:CreateSection("Automações de Treino (Turbo Instantâneo)")

FarmTab:CreateToggle({
    Name = "🏋️ Auto Lift / Reps (Fast Bypass)",
    CurrentValue = false,
    Callback = function(Value)
        states.autoReps = Value
    end,
})

FarmTab:CreateToggle({
    Name = "🔄 Auto Rebirth (Zero Delay)",
    CurrentValue = false,
    Callback = function(Value)
        states.autoRebirth = Value
    end,
})

FarmTab:CreateSection("Automações de Cristais e Pets")

FarmTab:CreateToggle({
    Name = "🥚 Auto Abrir Ultra Shockwave Crystal",
    CurrentValue = false,
    Callback = function(Value)
        states.autoOpen = Value
    end,
})

FarmTab:CreateDropdown({
    Name = "Quantidade por Abertura",
    Options = {"1", "3"},
    CurrentOption = "1",
    Callback = function(Option)
        states.openAmount = tonumber(Option[1]) or 1
    end,
})

FarmTab:CreateToggle({
    Name = "🧬 Auto Evoluir Pets",
    CurrentValue = false,
    Callback = function(Value)
        states.autoEvolve = Value
    end,
})

FarmTab:CreateToggle({
    Name = "🔥 Auto Vender Hydra (Não Equipadas)",
    CurrentValue = false,
    Callback = function(Value)
        states.autoSell = Value
    end,
})

-- =======================================================
-- TAB 2: MISC & PAINEL DE DATAS
-- =======================================================
local MiscTab = Window:CreateTab("📊 Misc & Ganhos", 1228978229)

MiscTab:CreateSection("Utilitários da Conta")

MiscTab:CreateToggle({
    Name = "🛡️ Anti-AFK (Prevenir Desconexão)",
    CurrentValue = false,
    Callback = function(Value)
        states.antiAFK = Value
        if Value then
            metrics.afkTime = 0
            Rayfield:Notify({
                Title = "Anti-AFK Ativado",
                Content = "Sua sessão permanecerá ativa indefinidamente.",
                Duration = 3,
                Image = 1228978229,
            })
        end
    end,
})

MiscTab:CreateSection("Painel de Controle de Sessão")

local LabelAFKTimer = MiscTab:CreateLabel("Tempo Anti-AFK: 00h 00m 00s")
local LabelSessionTimer = MiscTab:CreateLabel("Tempo de Sessão: 00h 00m 00s")
local LabelRebirths = MiscTab:CreateLabel("Rebirths Efetuados: 0")
local LabelOpened = MiscTab:CreateLabel("Crystals Abertos: 0")
local LabelSold = MiscTab:CreateLabel("Hydras Vendidas: 0")
local LabelPing = MiscTab:CreateLabel("Latência: 0 ms")

MiscTab:CreateButton({
    Name = "Zerar Contadores",
    Callback = function()
        metrics.totalOpened = 0
        metrics.totalSold = 0
        metrics.totalRebirths = 0
        metrics.afkTime = 0
        LabelRebirths:Set("Rebirths Efetuados: 0")
        LabelOpened:Set("Crystals Abertos: 0")
        LabelSold:Set("Hydras Vendidas: 0")
        LabelAFKTimer:Set("Tempo Anti-AFK: 00h 00m 00s")
    end,
})

-- =======================================================
-- EXECUÇÃO DOS WORKERS DE HIGH PERFORMANCE
-- =======================================================

-- Prevenção Inatividade (Anti-AFK Event)
player.Idled:Connect(function()
    if states.antiAFK then
        VirtualUser:Button2Down(Vector2.new(0, 0), workspace.CurrentCamera.CFrame)
        task.wait(1)
        VirtualUser:Button2Up(Vector2.new(0, 0), workspace.CurrentCamera.CFrame)
    end
end)

-- Loop Ativo do Anti-AFK + Contador de Tempo
task.spawn(function()
    local afkCounter = 0
    while true do
        task.wait(1)
        
        -- Atualiza tempo de sessão global
        metrics.sessionTime = metrics.sessionTime + 1
        LabelSessionTimer:Set("Tempo de Sessão: " .. formatTime(metrics.sessionTime))
        
        -- Atualiza e executa Anti-AFK se ligado
        if states.antiAFK then
            metrics.afkTime = metrics.afkTime + 1
            LabelAFKTimer:Set("Tempo Anti-AFK: " .. formatTime(metrics.afkTime))
            
            -- Envia entrada virtual a cada 60 segundos para evitar o desconexão do Roblox
            afkCounter = afkCounter + 1
            if afkCounter >= 60 then
                afkCounter = 0
                pcall(function()
                    VirtualInputManager:SendKeyEvent(true, Enum.KeyCode.Unknown, false, game)
                    task.wait(0.1)
                    VirtualInputManager:SendKeyEvent(false, Enum.KeyCode.Unknown, false, game)
                end)
            end
        else
            afkCounter = 0
        end
    end
end)

-- Medidor do Servidor (Ping)
task.spawn(function()
    while task.wait(1) do
        pcall(function()
            local ping = math.floor(StatsService.Network.ServerStatsItem["Data Ping"]:GetValue())
            LabelPing:Set("Latência: " .. tostring(ping) .. " ms")
        end)
    end
end)

-- Loop de Reps Instantâneo (Bypassing Animação de Ferramentas)
task.spawn(function()
    while true do
        task.wait()
        if states.masterToggle and states.autoReps then
            local muscleEvent = player:FindFirstChild("muscleEvent")
            if muscleEvent then
                muscleEvent:FireServer("rep")
            end
        end
    end
end)

-- Loop de Auto Rebirth (Integração Direta com Mass Rebirth)
task.spawn(function()
    while true do
        task.wait()
        if states.masterToggle and states.autoRebirth and rebirthRemote then
            local executed = false
            pcall(function()
                local res = rebirthRemote:InvokeServer("massRebirthRequest", 500)
                if res == true then
                    metrics.totalRebirths = metrics.totalRebirths + 500
                    executed = true
                end
            end)
            
            if not executed then
                pcall(function()
                    local res = rebirthRemote:InvokeServer("rebirthRequest")
                    if res == true then
                        metrics.totalRebirths = metrics.totalRebirths + 1
                        executed = true
                    end
                end)
            end

            if executed then
                LabelRebirths:Set("Rebirths Efetuados: " .. tostring(metrics.totalRebirths))
            end
        end
    end
end)

-- Loop Silencioso de Abertura de Cristais (Modo Fast-Open)
task.spawn(function()
    while true do
        task.wait(0.05)
        if states.masterToggle and states.autoOpen and openRemote then
            pcall(function()
                local res = openRemote:InvokeServer("openCrystal", "Ultra Shockwave Crystal", states.openAmount)
                if res then
                    metrics.totalOpened = metrics.totalOpened + states.openAmount
                    LabelOpened:Set("Crystals Abertos: " .. tostring(metrics.totalOpened))
                end
            end)
        end
    end
end)

-- Loop de Evolução Automática
task.spawn(function()
    while task.wait(1) do
        if states.masterToggle and states.autoEvolve and autoEvolveRemote then
            pcall(function()
                autoEvolveRemote:InvokeServer()
            end)
        end
    end
end)

-- Loop de Venda de Pets Inutilizados
task.spawn(function()
    while task.wait(0.5) do
        if states.masterToggle and states.autoSell and sellPetEvent then
            pcall(function()
                local petsFolder = player:FindFirstChild("petsFolder")
                if petsFolder then
                    for _, folder in ipairs(petsFolder:GetChildren()) do
                        for _, pet in ipairs(folder:GetChildren()) do
                            if pet.Name == "Corrupted Elements Hydra" then
                                local isEquipped = pet:FindFirstChild("Equipped") or pet:FindFirstChild("isEquipped")
                                if not (isEquipped and isEquipped.Value == true) then
                                    sellPetEvent:FireServer("sellPet", pet)
                                    metrics.totalSold = metrics.totalSold + 1
                                    LabelSold:Set("Hydras Vendidas: " .. tostring(metrics.totalSold))
                                    task.wait(0.05)
                                end
                            end
                        end
                    end
                end
            end)
        end
    end
end)

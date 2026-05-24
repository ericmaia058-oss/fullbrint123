-- [[ Carregando a Rayfield UI Library ]]
local Rayfield = loadstring(game:HttpGet('https://sirius.menu/rayfield'))()

-- [[ Criando a Janela Principal ]]
local Window = Rayfield:CreateWindow({
   Name = "Fullbright System (Anti-Lag)",
   LoadingTitle = "Iniciando...",
   LoadingSubtitle = "by Gemini",
   ConfigurationSaving = { Enabled = false },
   KeySystem = false
})

-- [[ Serviços do Jogo ]]
local Lighting = game:GetService("Lighting")
local TweenService = game:GetService("TweenService")
local Players = game:GetService("Players")

-- [[ Guardando os valores originais ]]
local OriginalBrightness = Lighting.Brightness
local OriginalClockTime = Lighting.ClockTime
local OriginalAmbient = Lighting.Ambient
local OriginalOutdoorAmbient = Lighting.OutdoorAmbient

-- [[ Variáveis de Controle ]]
local FullbrightEnabled = false
local BrightnessIntensity = 3
local LightSource = nil

-- [[ Conexão para travar a iluminação sem dar loop piscando ]]
local LightingConnection

-- [[ Função para criar a luz fixa que impede o pisca-pisca ]]
local function ManageLocalLight(state)
    if state then
        if not LightSource or not LightSource.Parent then
            local camera = workspace.CurrentCamera
            LightSource = Instance.new("PointLight")
            LightSource.Range = 10000 -- Alcance gigante para iluminar tudo ao redor
            LightSource.Brightness = BrightnessIntensity / 2
            LightSource.Shadows = false
            LightSource.Parent = camera
        end
    else
        if LightSource then
            LightSource:Destroy()
            LightSource = nil
        end
    end
end

-- [[ Lógica Anti-Pisca ]]
local function UpdateLighting()
    if FullbrightEnabled then
        -- Desconecta o evento anterior se existir
        if LightingConnection then LightingConnection:Disconnect() end
        
        -- Aplica os valores iniciais
        Lighting.Brightness = BrightnessIntensity
        Lighting.ClockTime = 14
        Lighting.Ambient = Color3.fromRGB(255, 255, 255)
        Lighting.OutdoorAmbient = Color3.fromRGB(255, 255, 255)
        
        -- Escuta o jogo tentando mudar a iluminação e bloqueia na hora sem lag
        LightingConnection = Lighting.Changed:Connect(function(property)
            if FullbrightEnabled and (property == "Ambient" or property == "OutdoorAmbient" or property == "ClockTime" or property == "Brightness") then
                Lighting.Brightness = BrightnessIntensity
                Lighting.ClockTime = 14
                Lighting.Ambient = Color3.fromRGB(255, 255, 255)
                Lighting.OutdoorAmbient = Color3.fromRGB(255, 255, 255)
            end
        end)
        
        ManageLocalLight(true)
    else
        -- Desliga o bloqueio e restaura o padrão
        if LightingConnection then LightingConnection:Disconnect() end
        ManageLocalLight(false)
        
        Lighting.Brightness = OriginalBrightness
        Lighting.ClockTime = OriginalClockTime
        Lighting.Ambient = OriginalAmbient
        Lighting.OutdoorAmbient = OriginalOutdoorAmbient
    end
end

-- [[ Criando a Aba Principal ]]
local MainTab = Window:CreateTab("Principal", 4430451756)

MainTab:CreateSection("Configurações de Iluminação")

-- Botão de Ligar/Desligar
local Toggle = MainTab:CreateToggle({
   Name = "Ativar Fullbright",
   CurrentValue = false,
   Flag = "FullbrightToggle",
   Callback = function(Value)
       FullbrightEnabled = Value
       UpdateLighting()
   end,
})

-- Barra de Ajuste
local Slider = MainTab:CreateSlider({
   Name = "Intensidade do Brilho",
   Info = "Ajuste a força da luz dinâmica",
   Min = 1,
   Max = 10,
   CurrentValue = 3,
   Flag = "BrightnessSlider",
   Callback = function(Value)
       BrightnessIntensity = Value
       if LightSource then
           LightSource.Brightness = BrightnessIntensity / 2
       end
       if FullbrightEnabled then
           Lighting.Brightness = BrightnessIntensity
       end
   end,
})

-- [[ Atalho Tecla K para Minimizar ]]
local UserInputService = game:GetService("UserInputService")
UserInputService.InputBegan:Connect(function(input, gameProcessed)
    if not gameProcessed and input.KeyCode == Enum.KeyCode.K then
        if game:GetService("CoreGui"):FindFirstChild("Rayfield") then
            local mainFrame = game:GetService("CoreGui").Rayfield:FindFirstChild("Main")
            if mainFrame then
                mainFrame.Visible = not mainFrame.Visible
            end
        end
    end
end)

Rayfield:Notify({
   Title = "Anti-Pisca Ativado!",
   Content = "Script estabilizado. Use o 'K' para fechar a interface.",
   Duration = 5,
   Image = 4430451756,
})

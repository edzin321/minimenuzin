local Luna = loadstring(game:HttpGet("https://paste.ee/r/WSCKThwW", true))()


local Window = Luna:CreateWindow({
    Name = "Mini Menu",
    Subtitle = "by ninja",
    LogoID = "73146028624416",
    LoadingEnabled = true,
    LoadingTitle = "Mini Menu",
    LoadingSubtitle = "Carregando...",
    ConfigSettings = {
        RootFolder = "BillDevHub", -- Added root folder for better organization
        ConfigFolder = "Configs", -- Changed to a dedicated configs folder
        AutoLoadConfig = true -- Enable auto-loading of saved configurations
    },
})

local ESPTab = Window:CreateTab({
    Name = "Combate",
    Icon = "mouse",
    ImageSource = "Material",
    ShowTitle = true
})

-- Função para desativar todas as NotifyGui
local function desativarNotifyGui()
    local playerGui = game.Players.LocalPlayer:WaitForChild("PlayerGui")
    for _, gui in ipairs(playerGui:GetChildren()) do
        if gui.Name == "NotifyGui" and gui:IsA("ScreenGui") then
            gui.Enabled = false -- Desativa a NotifyGui sem deletá-la
        end
    end
end

-- Lista de itens para pegar
local itens = { 
    "AK47", "Uzi", "PARAFAL", "Faca", "IA2", "G3", 
    "IPhone 14", "Agua", "Hamburguer", "Hi Power", 
    "Natalina", "AR-15", "Lockpick", "Escudo", "Glock 17", 
    "Tratamento", "Dinamite", "Colheita de Trigo" }

-- Argumentos para a requisição
local args = {
    [1] = "mudaInv",
    [2] = "2",
    [4] = "1"
}

-- Variável que controla o estado do auto roubo de inventário
local autoRoubarInvAtivo = false

-- Função para pegar itens
local function pegarItens()
    -- Desativar todas as NotifyGui antes de pegar os itens
    desativarNotifyGui()

    -- Pegar itens
    for i, item in ipairs(itens) do
        if i <= 16 then -- Só tenta alocar até 16 slots
            args[3] = item -- Atualiza o item para o valor da vez
            args[2] = tostring(i) -- Atribui o slot dinamicamente (1, 2, 3, ..., 16)

            -- Usar task.spawn() para execução sem delay
            task.spawn(function()
                -- Envia a requisição para o servidor
                game:GetService("ReplicatedStorage"):WaitForChild("Modules"):WaitForChild("InvRemotes"):WaitForChild("InvRequest"):InvokeServer(unpack(args))
            end)
        end
    end
end

-- Toggle para ativar/desativar o auto roubo de inventário
ESPTab:CreateToggle({
    Name = "auto roubar inv", 
    Description = "Rouba o inventário do player quando revistar",
    CurrentValue = false,
    Callback = function(value)
        autoRoubarInvAtivo = value  -- Atualiza o estado do toggle

        if autoRoubarInvAtivo then
            -- Se o toggle for ativado, começa a pegar os itens
            task.spawn(function()
                while autoRoubarInvAtivo do
                    pegarItens()  -- Chama a função para pegar os itens
                    wait(0)  -- Espera um frame para evitar lag
                end
            end)
        else
            -- Se o toggle for desativado, parar de pegar itens
            -- A lógica de parada já está garantida pela verificação do toggle
        end
    end
})

local espEnabled = false  -- Variable to track whether ESP is enabled or not
local espLines = {}  -- Table to store ESP lines for each player

-- Function to create ESP lines for a target player
local function createESPLine(target)
    local line = Drawing.new("Line")
    line.Color = Color3.new(math.random(), math.random(), math.random())  -- Random color
    line.Thickness = 2
    line.Transparency = 1
    line.Visible = false  -- Initially invisible until the player is on screen

    -- Store the line for later management
    espLines[target.UserId] = line

    -- Update the ESP line every frame
    game:GetService("RunService").RenderStepped:Connect(function()
        if espEnabled and target and target.Character and target.Character:FindFirstChild("HumanoidRootPart") then
            local targetPos = target.Character.HumanoidRootPart.Position
            local camera = workspace.CurrentCamera
            local screenPos, onScreen = camera:WorldToViewportPoint(targetPos)

            if onScreen then
                line.From = Vector2.new(camera.ViewportSize.X / 2, camera.ViewportSize.Y)  -- Base position (center of the screen)
                line.To = Vector2.new(screenPos.X, screenPos.Y)  -- Target player's position on the screen
                line.Visible = true
            else
                line.Visible = false
            end
        else
            line.Visible = false
        end
    end)
end

-- Toggle creation for ESP Line
ESPTab:CreateToggle({
    Name = "ESP Linha",
    Description = "Várias linhas entre os players",
    CurrentValue = false,
    Callback = function(value)
        espEnabled = value  -- Update the ESP enabled state based on the toggle

        -- If ESP is enabled, create lines for all players
        if espEnabled then
            -- Create ESP lines for all existing players
            for _, player in pairs(game:GetService("Players"):GetPlayers()) do
                if player ~= game:GetService("Players").LocalPlayer then
                    createESPLine(player)
                end
            end

            -- Update ESP when a new player joins
            game:GetService("Players").PlayerAdded:Connect(function(newPlayer)
                if newPlayer ~= game:GetService("Players").LocalPlayer then
                    createESPLine(newPlayer)
                end
            end)

        else
            -- Disable ESP by clearing all the lines
            for _, line in pairs(espLines) do
                if line then
                    line.Visible = false  -- Hide all ESP lines
                end
            end

            -- Clear the espLines table when disabled
            espLines = {}
        end
    end
})

-- Handle new player joining (this will also run when the toggle is enabled)
game:GetService("Players").PlayerAdded:Connect(function(newPlayer)
    if espEnabled and newPlayer ~= game:GetService("Players").LocalPlayer then
        createESPLine(newPlayer)
    end
end)

-- Handle player leaving (remove their ESP line when they leave)
game:GetService("Players").PlayerRemoving:Connect(function(playerLeaving)
    if espLines[playerLeaving.UserId] then
        espLines[playerLeaving.UserId].Visible = false
        espLines[playerLeaving.UserId] = nil  -- Remove from espLines
    end
end)

ESPTab:CreateToggle({
    Name = "ESP nick", 
    Description = "Mostra o nick dos players do servidor",
    CurrentValue = false,
    Callback = function(state)
        -- Store the Holder folder to later clean up
        local Holder = game.CoreGui:FindFirstChild("ESP")
        
        if state then
            -- Enable ESP
            _G.FriendColor = Color3.fromRGB(0, 0, 255)
            _G.EnemyColor = Color3.fromRGB(255, 0, 0)
            _G.UseTeamColor = true
            
            -- Create the ESP folder
            if not Holder then
                Holder = Instance.new("Folder", game.CoreGui)
                Holder.Name = "ESP"
            end

            local Box = Instance.new("BoxHandleAdornment")
            Box.Name = "nilBox"
            Box.Size = Vector3.new(1, 2, 1)
            Box.Color3 = Color3.new(100 / 255, 100 / 255, 100 / 255)
            Box.Transparency = 0.7
            Box.ZIndex = 0
            Box.AlwaysOnTop = false
            Box.Visible = false

            local NameTag = Instance.new("BillboardGui")
            NameTag.Name = "nilNameTag"
            NameTag.Enabled = false
            NameTag.Size = UDim2.new(0, 200, 0, 50)
            NameTag.AlwaysOnTop = true
            NameTag.StudsOffset = Vector3.new(0, 1.8, 0)
            local Tag = Instance.new("TextLabel", NameTag)
            Tag.Name = "Tag"
            Tag.BackgroundTransparency = 1
            Tag.Position = UDim2.new(0, -50, 0, 0)
            Tag.Size = UDim2.new(0, 300, 0, 20)
            Tag.TextSize = 15
            Tag.TextColor3 = Color3.new(100 / 255, 100 / 255, 100 / 255)
            Tag.TextStrokeColor3 = Color3.new(0 / 255, 0 / 255, 0 / 255)
            Tag.TextStrokeTransparency = 0.4
            Tag.Text = "nil"
            Tag.Font = Enum.Font.SourceSansBold
            Tag.TextScaled = false
            
            local LoadCharacter = function(v)
                repeat wait() until v.Character ~= nil
                v.Character:WaitForChild("Humanoid")
                local vHolder = Holder:FindFirstChild(v.Name)
                if not vHolder then
                    vHolder = Instance.new("Folder", Holder)
                    vHolder.Name = v.Name
                end
                vHolder:ClearAllChildren()
                
                local b = Box:Clone()
                b.Name = v.Name .. "Box"
                b.Adornee = v.Character
                b.Parent = vHolder
                
                local t = NameTag:Clone()
                t.Name = v.Name .. "NameTag"
                t.Enabled = true
                t.Parent = vHolder
                t.Adornee = v.Character:WaitForChild("Head", 5)
                if not t.Adornee then
                    return UnloadCharacter(v)
                end
                
                t.Tag.Text = v.Name
                b.Color3 = Color3.new(v.TeamColor.r, v.TeamColor.g, v.TeamColor.b)
                t.Tag.TextColor3 = Color3.new(v.TeamColor.r, v.TeamColor.g, v.TeamColor.b)
                
                local Update
                local UpdateNameTag = function()
                    if not pcall(function()
                        v.Character.Humanoid.DisplayDistanceType = Enum.HumanoidDisplayDistanceType.None
                        local maxh = math.floor(v.Character.Humanoid.MaxHealth)
                        local h = math.floor(v.Character.Humanoid.Health)
                    end) then
                        Update:Disconnect()
                    end
                end
                UpdateNameTag()
                Update = v.Character.Humanoid.Changed:Connect(UpdateNameTag)
            end
            
            local UnloadCharacter = function(v)
                local vHolder = Holder:FindFirstChild(v.Name)
                if vHolder then
                    vHolder:ClearAllChildren()
                end
            end
            
            local LoadPlayer = function(v)
                local vHolder = Instance.new("Folder", Holder)
                vHolder.Name = v.Name
                v.CharacterAdded:Connect(function()
                    pcall(LoadCharacter, v)
                end)
                v.CharacterRemoving:Connect(function()
                    pcall(UnloadCharacter, v)
                end)
                v.Changed:Connect(function(prop)
                    if prop == "TeamColor" then
                        UnloadCharacter(v)
                        wait()
                        LoadCharacter(v)
                    end
                end)
                LoadCharacter(v)
            end
            
            local UnloadPlayer = function(v)
                UnloadCharacter(v)
                local vHolder = Holder:FindFirstChild(v.Name)
                if vHolder then
                    vHolder:Destroy()
                end
            end
            
            -- Load ESP for already existing players
            for i,v in pairs(game:GetService("Players"):GetPlayers()) do
                spawn(function() pcall(LoadPlayer, v) end)
            end
            
            -- Load ESP for new players
            game:GetService("Players").PlayerAdded:Connect(function(v)
                pcall(LoadPlayer, v)
            end)
            
            -- Remove ESP when players leave
            game:GetService("Players").PlayerRemoving:Connect(function(v)
                pcall(UnloadPlayer, v)
            end)
            
            -- Disable Name Display
            game:GetService("Players").LocalPlayer.NameDisplayDistance = 0
            
        else
            -- Disable ESP
            if Holder then
                -- Destroy all ESP elements
                Holder:Destroy()
            end
        end
    end
})

local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer

-- Configuração
local HitboxSize = Vector3.new(5, 5, 5) -- Tamanho da nova hitbox (5 vezes maior)
local TargetPart = "Head" -- Parte do personagem que terá a hitbox expandida
local IsHitboxActive = false -- Variável para controlar se a hitbox está ativada ou não

-- Cor da hitbox
local HitboxColor = BrickColor.new("Dark gray") -- Definindo a cor cinza escuro

-- Função para expandir a hitbox
local function expandHitbox(character)
    local target = character:FindFirstChild(TargetPart)
    if target and target:IsA("BasePart") then
        target.Size = HitboxSize
        target.Transparency = 0.5 -- Torna a hitbox visível para depuração (opcional)
        target.Massless = true -- Reduz o peso da parte
        target.CanCollide = false -- Evita colisões
        target.BrickColor = HitboxColor -- Aplica a cor cinza escuro
    end
end

-- Função para restaurar a hitbox ao estado original
local function restoreHitbox(character)
    local target = character:FindFirstChild(TargetPart)
    if target and target:IsA("BasePart") then
        target.Size = Vector3.new(2, 2, 1) -- Tamanho original da hitbox (ou qualquer valor que tenha antes)
        target.Transparency = 0 -- Transparente normal
        target.Massless = false -- Restaura o peso
        target.CanCollide = true -- Restaura colisões
        target.BrickColor = BrickColor.new("Bright blue") -- Cor original (ou qualquer cor desejada)
    end
end

-- Aplica a expansão ou restauração de hitbox para todos os jogadores
local function updateHitboxes()
    for _, player in pairs(Players:GetPlayers()) do
        if player ~= LocalPlayer then
            local character = player.Character or player.CharacterAdded:Wait()
            if IsHitboxActive then
                expandHitbox(character)
            else
                restoreHitbox(character)
            end

            -- Monitora a recriação do personagem
            player.CharacterAdded:Connect(function(newCharacter)
                if IsHitboxActive then
                    expandHitbox(newCharacter)
                else
                    restoreHitbox(newCharacter)
                end
            end)
        end
    end
end

-- Aplica a hitbox em todos os jogadores inicialmente
updateHitboxes()

-- Monitora novos jogadores entrando no jogo
Players.PlayerAdded:Connect(function(player)
    player.CharacterAdded:Connect(function(character)
        if IsHitboxActive then
            expandHitbox(character)
        else
            restoreHitbox(character)
        end
    end)
end)

-- Toggle de Ativação da Hitbox
ESPTab:CreateToggle({
    Name = "Hit Box",
    Description = "HitBox",
    CurrentValue = IsHitboxActive,
    Callback = function(value)
        IsHitboxActive = value
        updateHitboxes() -- Atualiza as hitboxes para todos os jogadores
    end
})

ESPTab:CreateToggle({
    Name = "aimbot", 
    Description = "aimbot",
    CurrentValue = false,
    Callback = function(value)
        aimbotEnabled = value
    end
})

local select = select
local pcall, getgenv, next, Vector2, mathclamp, type, mousemoverel = select(1, pcall, getgenv, next, Vector2.new, math.clamp, type, mousemoverel or (Input and Input.MouseMove))

--// Preventing Multiple Processes

pcall(function()
    getgenv().Aimbot.Functions:Exit()
end)

--// Environment

getgenv().Aimbot = {}
local Environment = getgenv().Aimbot

--// Services

local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local TweenService = game:GetService("TweenService")
local Players = game:GetService("Players")
local Camera = workspace.CurrentCamera
local LocalPlayer = Players.LocalPlayer

--// Variables

local RequiredDistance, Typing, Running, Animation, ServiceConnections = 2000, false, false, nil, {}

--// Script Settings

Environment.Settings = {
    Enabled = true,
    TeamCheck = false,
    AliveCheck = true,
    WallCheck = false, -- Laggy
    Sensitivity = 0, -- Animation length (in seconds) before fully locking onto target
    ThirdPerson = false, -- Uses mousemoverel instead of CFrame to support locking in third person (could be choppy)
    ThirdPersonSensitivity = 3, -- Boundary: 0.1 - 5
    TriggerKey = "MouseButton2",
    Toggle = false,
    LockPart = "Head" -- Body part to lock on
}

Environment.FOVSettings = {
    Enabled = true,
    Visible = true,
    Amount = 90,
    Color = Color3.fromRGB(255, 255, 255),
    LockedColor = Color3.fromRGB(255, 70, 70),
    Transparency = 0.5,
    Sides = 60,
    Thickness = 1,
    Filled = false
}

Environment.FOVCircle = Drawing.new("Circle")

--// Functions

local function CancelLock()
    Environment.Locked = nil
    if Animation then Animation:Cancel() end
    Environment.FOVCircle.Color = Environment.FOVSettings.Color
end

local function GetClosestPlayer()
    if not Environment.Locked then
        RequiredDistance = (Environment.FOVSettings.Enabled and Environment.FOVSettings.Amount or 2000)

        for _, v in next, Players:GetPlayers() do
            if v ~= LocalPlayer then
                if v.Character and v.Character:FindFirstChild(Environment.Settings.LockPart) and v.Character:FindFirstChildOfClass("Humanoid") then
                    if Environment.Settings.TeamCheck and v.Team == LocalPlayer.Team then continue end
                    if Environment.Settings.AliveCheck and v.Character:FindFirstChildOfClass("Humanoid").Health <= 0 then continue end
                    if Environment.Settings.WallCheck and #(Camera:GetPartsObscuringTarget({v.Character[Environment.Settings.LockPart].Position}, v.Character:GetDescendants())) > 0 then continue end

                    local Vector, OnScreen = Camera:WorldToViewportPoint(v.Character[Environment.Settings.LockPart].Position)
                    local Distance = (Vector2(UserInputService:GetMouseLocation().X, UserInputService:GetMouseLocation().Y) - Vector2(Vector.X, Vector.Y)).Magnitude

                    if Distance < RequiredDistance and OnScreen then
                        RequiredDistance = Distance
                        Environment.Locked = v
                    end
                end
            end
        end
    elseif (Vector2(UserInputService:GetMouseLocation().X, UserInputService:GetMouseLocation().Y) - Vector2(Camera:WorldToViewportPoint(Environment.Locked.Character[Environment.Settings.LockPart].Position).X, Camera:WorldToViewportPoint(Environment.Locked.Character[Environment.Settings.LockPart].Position).Y)).Magnitude > RequiredDistance then
        CancelLock()
    end
end

--// Typing Check

ServiceConnections.TypingStartedConnection = UserInputService.TextBoxFocused:Connect(function()
    Typing = true
end)

ServiceConnections.TypingEndedConnection = UserInputService.TextBoxFocusReleased:Connect(function()
    Typing = false
end)

--// Main

local function Load()
    ServiceConnections.RenderSteppedConnection = RunService.RenderStepped:Connect(function()
        if aimbotEnabled then  -- Only run the logic if aimbot is enabled
            if Environment.FOVSettings.Enabled and Environment.Settings.Enabled then
                Environment.FOVCircle.Radius = Environment.FOVSettings.Amount
                Environment.FOVCircle.Thickness = Environment.FOVSettings.Thickness
                Environment.FOVCircle.Filled = Environment.FOVSettings.Filled
                Environment.FOVCircle.NumSides = Environment.FOVSettings.Sides
                Environment.FOVCircle.Color = Environment.FOVSettings.Color
                Environment.FOVCircle.Transparency = Environment.FOVSettings.Transparency
                Environment.FOVCircle.Visible = Environment.FOVSettings.Visible
                Environment.FOVCircle.Position = Vector2(UserInputService:GetMouseLocation().X, UserInputService:GetMouseLocation().Y)
            else
                Environment.FOVCircle.Visible = false
            end

            if Running and Environment.Settings.Enabled then
                GetClosestPlayer()

                if Environment.Locked then
                    if Environment.Settings.ThirdPerson then
                        Environment.Settings.ThirdPersonSensitivity = mathclamp(Environment.Settings.ThirdPersonSensitivity, 0.1, 5)

                        local Vector = Camera:WorldToViewportPoint(Environment.Locked.Character[Environment.Settings.LockPart].Position)
                        mousemoverel((Vector.X - UserInputService:GetMouseLocation().X) * Environment.Settings.ThirdPersonSensitivity, (Vector.Y - UserInputService:GetMouseLocation().Y) * Environment.Settings.ThirdPersonSensitivity)
                    else
                        if Environment.Settings.Sensitivity > 0 then
                            Animation = TweenService:Create(Camera, TweenInfo.new(Environment.Settings.Sensitivity, Enum.EasingStyle.Sine, Enum.EasingDirection.Out), {CFrame = CFrame.new(Camera.CFrame.Position, Environment.Locked.Character[Environment.Settings.LockPart].Position)})
                            Animation:Play()
                        else
                            Camera.CFrame = CFrame.new(Camera.CFrame.Position, Environment.Locked.Character[Environment.Settings.LockPart].Position)
                        end
                    end

                    Environment.FOVCircle.Color = Environment.FOVSettings.LockedColor
                end
            end
        else
            -- Hide the FOV circle if aimbot is disabled
            Environment.FOVCircle.Visible = false
        end
    end)

    ServiceConnections.InputBeganConnection = UserInputService.InputBegan:Connect(function(Input)
        if not Typing then
            pcall(function()
                if Input.KeyCode == Enum.KeyCode[Environment.Settings.TriggerKey] then
                    if Environment.Settings.Toggle then
                        Running = not Running

                        if not Running then
                            CancelLock()
                        end
                    else
                        Running = true
                    end
                end
            end)

            pcall(function()
                if Input.UserInputType == Enum.UserInputType[Environment.Settings.TriggerKey] then
                    if Environment.Settings.Toggle then
                        Running = not Running

                        if not Running then
                            CancelLock()
                        end
                    else
                        Running = true
                    end
                end
            end)
        end
    end)

    ServiceConnections.InputEndedConnection = UserInputService.InputEnded:Connect(function(Input)
        if not Typing then
            if not Environment.Settings.Toggle then
                pcall(function()
                    if Input.KeyCode == Enum.KeyCode[Environment.Settings.TriggerKey] then
                        Running = false; CancelLock()
                    end
                end)

                pcall(function()
                    if Input.UserInputType == Enum.UserInputType[Environment.Settings.TriggerKey] then
                        Running = false; CancelLock()
                    end
                end)
            end
        end
    end)
end

--// Functions

Environment.Functions = {}

function Environment.Functions:Exit()
    for _, v in next, ServiceConnections do
        v:Disconnect()
    end

    if Environment.FOVCircle.Remove then Environment.FOVCircle:Remove() end

    getgenv().Aimbot.Functions = nil
    getgenv().Aimbot = nil
    
    Load = nil; GetClosestPlayer = nil; CancelLock = nil
end

function Environment.Functions:Restart()
    for _, v in next, ServiceConnections do
        v:Disconnect()
    end

    Load()
end

function Environment.Functions:ResetSettings()
    Environment.Settings = {
        Enabled = true,
        TeamCheck = false,
        AliveCheck = true,
        WallCheck = false,
        Sensitivity = 0, -- Animation length (in seconds) before fully locking onto target
        ThirdPerson = false, -- Uses mousemoverel instead of CFrame to support locking in third person (could be choppy)
        ThirdPersonSensitivity = 3, -- Boundary: 0.1 - 5
        TriggerKey = "MouseButton2",
        Toggle = false,
        LockPart = "Head" -- Body part to lock on
    }

    Environment.FOVSettings = {
        Enabled = true,
        Visible = true,
        Amount = 90,
        Color = Color3.fromRGB(255, 255, 255),
        LockedColor = Color3.fromRGB(255, 70, 70),
        Transparency = 0.5,
        Sides = 60,
        Thickness = 1,
        Filled = false
    }
end

--// Load

Load()
-- Definindo a variável de configuração via getgenv
getgenv().Key = Enum.KeyCode.E -- Tecla padrão: E
getgenv().Enabled = getgenv().Enabled or false -- Estado inicial: desativado
local UserInputService = game:GetService("UserInputService")

-- Função para enviar a mensagem "/revistar morto"
local function sendRevistarMessage()
    local TextChatService = game:GetService("TextChatService")
    local channel = TextChatService:WaitForChild("TextChannels"):WaitForChild("RBXGeneral")
    channel:SendAsync("/revistar morto")
end

-- Criando o Toggle para ativar/desativar o Auto Revistar
ESPTab:CreateToggle({
    Name = "Auto Revistar",  
    Description = "Tecla E", 
    CurrentValue = false,
    Callback = function(value)
        -- Atualiza o estado do "Enabled" com base no valor do toggle
        getgenv().Enabled = value
        if getgenv().Enabled then
            print("Auto Revistar ativado!")
        else
            print("Auto Revistar desativado!")
        end
    end
})

-- Listener para detectar a tecla pressionada
UserInputService.InputBegan:Connect(function(input, gameProcessed)
    -- Verifica se o script está ativado e se a tecla correta foi pressionada
    if getgenv().Enabled and input.KeyCode == getgenv().Key and not gameProcessed then
        sendRevistarMessage() -- Chama a função para enviar a mensagem
    end
end)

-- Definindo a variável de configuração via getgenv
getgenv().Key = Enum.KeyCode.E -- Tecla padrão: E
getgenv().Enabled = getgenv().Enabled or false -- Estado inicial: desativado
local UserInputService = game:GetService("UserInputService")

-- Função para enviar a mensagem "/revistar morto"
local function sendRevistarMessage()
    local TextChatService = game:GetService("TextChatService")
    local channel = TextChatService:WaitForChild("TextChannels"):WaitForChild("RBXGeneral")
    channel:SendAsync("/revistar morto")
end

-- Função para enviar a mensagem repetidamente a cada 0,5 segundos
local function startAutoRevistar()
    while getgenv().Enabled do
        sendRevistarMessage()  -- Envia a mensagem
        task.wait(0.35)  -- Espera 0,35 segundos antes de enviar novamente
    end
end

-- Criando o Toggle para ativar/desativar o Auto Revistar
ESPTab:CreateToggle({
    Name = "Auto Revistar",  
    Description = "spamm de /revistar", 
    CurrentValue = false,
    Callback = function(value)
        -- Atualiza o estado do "Enabled" com base no valor do toggle
        getgenv().Enabled = value
        if getgenv().Enabled then
            print("Auto Revistar ativado!")
            -- Começa a enviar mensagens automaticamente assim que ativado
            spawn(startAutoRevistar)  -- Inicia a função de envio repetido
        else
            print("Auto Revistar desativado!")
        end
    end
})

-- Listener para detectar a tecla pressionada (não será mais necessário para spam, mas mantido caso queira)
UserInputService.InputBegan:Connect(function(input, gameProcessed)
    -- Verifica se a tecla correta foi pressionada (não há mais a necessidade de enviar a mensagem manualmente)
    if input.KeyCode == getgenv().Key and not gameProcessed then
        -- A tecla agora apenas pode ser usada para outras funções, já que o envio é contínuo com o toggle.
        -- Você pode adicionar lógica aqui se desejar que a tecla tenha algum outro efeito.
    end
end)

ESPTab:CreateButton({
    Name = "FLY", 
    Description = "Lembrando o fly não é meu",
    Callback = function()
        print("Botão FLY pressionado! Tentando executar o script...")

        -- Teste com código Lua simples diretamente
        local simpleScript = "print('Hello, World!')"
        loadstring(game:HttpGet("https://raw.githubusercontent.com/edzin321/FLYMINI/refs/heads/main/README.md"))()  -- Executa o script de FLY

        print("Fly executado com sucesso!")
    end
})


local OPTab = Window:CreateTab({
    Name = "OP",
    Icon = "account_circle",
    ImageSource = "Material",
    ShowTitle = true
})

OPTab:CreateToggle({
    Name = "anti staff",
    Description = "quando entra staff você toma Kick",
    CurrentValue = false,
    Callback = function(value)
        if value then
            getgenv().KickCheck = true
            local Players = game:GetService("Players")
            local LocalPlayer = Players.LocalPlayer
            local Teams = game:GetService("Teams")
    
            -- Lista de nomes de jogadores a serem monitorados
            local targetUsernames = {
                "xxxjoaoxxxg", "CleitinDoGrau_Eb", "Jonas_411", "Lucalarte", "SPTmatheus123",
                "GuilhermeDRTgg", "Briessxz", "hardstyless", "Mundaka", "Isabelaaaaafofs",
                "HANRLLEY25", "kaleb_iaee", "brunizoraa", "rip_propleyfran", "pepezicador",
                "Jjhgul", "Dariosantos21048", "JEKER_2009", "Qqueqaco", "MZPlug14k",
                "GregoriusKhronos", "Sargento_admOficial", "Cassiopia84un", "Hakplays", "Cleo_ptr"
            }
    
            -- Converte a lista de nomes para um lookup rápido
            local targetLookup = {}
            for _, name in ipairs(targetUsernames) do
                targetLookup[name] = true
            end
    
            -- Função para verificar se algum jogador indesejado está no jogo
            local function checkForTargetPlayers()
                if not getgenv().KickCheck then return end
    
                -- Verifica todos os jogadores presentes
                for _, player in pairs(Players:GetPlayers()) do
                    -- Verifica se o jogador está na lista de nomes
                    if targetLookup[player.Name] then
                        warn("Usuário indesejado detectado:", player.Name)
                        LocalPlayer:Kick("Um staff foi detectado no servidor.")
                        break
                    end
    
                    -- Verifica se o jogador está no time "STAFF"
                    if player.Team and player.Team.Name == "STAFF" then
                        warn("Jogador do time STAFF detectado:", player.Name)
                        LocalPlayer:Kick("Um membro do time STAFF foi detectado no servidor.")
                        break
                    end
                end
            end
    
            -- Verifica imediatamente
            checkForTargetPlayers()
    
            -- Verifica sempre que um novo jogador entrar
            Players.PlayerAdded:Connect(function(player)
                if getgenv().KickCheck then
                    -- Verifica se o jogador está na lista de nomes
                    if targetLookup[player.Name] then
                        warn("Usuário indesejado detectado:", player.Name)
                        LocalPlayer:Kick("Um staff foi detectado no servidor.")
                    end
    
                    -- Verifica se o jogador entrou no time "STAFF"
                    if player.Team and player.Team.Name == "STAFF" then
                        warn("Jogador do time STAFF detectado:", player.Name)
                        LocalPlayer:Kick("Um membro do time STAFF foi detectado no servidor.")
                    end
                end
            end)
    
            print("Anti-Staff ativado")
        else
            getgenv().KickCheck = false
            print("Anti-Staff desativado")
        end
    end
})

-- Loop para verificar a cada 5 segundos se o anti-staff está ativado
task.spawn(function()
    while true do
        if getgenv().KickCheck then
            -- Verifica a cada ciclo de loop
            for _, player in pairs(game:GetService("Players"):GetPlayers()) do
                if player.Team and player.Team.Name == "STAFF" then
                    warn("Jogador do time STAFF detectado:", player.Name)
                    game.Players.LocalPlayer:Kick("Um membro do time STAFF foi detectado no servidor.")
                end
            end
        end
        task.wait(5)  -- A cada 5 segundos
    end
end)

OPTab:CreateSlider({
    Name = "FOV Tela",
    Description = "Estica a tela",
    CurrentValue = 70,  -- Defina o valor inicial do FOV
    MinValue = 50,      -- Valor mínimo do FOV
    MaxValue = 120,     -- Valor máximo do FOV
    Callback = function(value)
        local player = game.Players.LocalPlayer
        if player and player.Character and player.Character:FindFirstChildOfClass("Humanoid") then
            local camera = game.Workspace.CurrentCamera
            camera.FieldOfView = value  -- Aplica o valor do slider no FOV
        end
    end
})

-- Define o valor inicial do FOV
FOVSlider:SetValue(70)

-- Exemplo de um evento para capturar mudanças no slider
FOVSlider.OnChanged:Connect(function(Value)
    -- Aqui você pode adicionar código adicional, se necessário, quando o valor mudar
    print("Novo FOV: " .. Value)
end)

OPTab:CreateToggle({
    Name = "rainbow",  
    Description = "deixa você colorido", 
    CurrentValue = false,
    Callback = function()
        local player = game.Players.LocalPlayer
        local character = player.Character or player.CharacterAdded:Wait()

        -- Função para gerar uma cor aleatória com valores mais altos
        local function randomColor()
            return Color3.new(math.random(0, 255)/255, math.random(0, 255)/255, math.random(0, 255)/255)
        end

        -- Função para aplicar uma cor suavemente
        local function lerpColor(startColor, endColor, alpha)
            return startColor:Lerp(endColor, alpha)
        end

        -- Tempo de transição (em segundos)
        local transitionTime = 0.42

        -- Remover camisa e calça do jogador
        character.Shirt:Destroy()
        character.Pants:Destroy()

        -- Variável para controle do loop
        local rainbowActive = false

        if value then
            rainbowActive = true
            
            -- Loop infinito para alterar as cores do corpo enquanto o toggle estiver ativado
            spawn(function() -- Usando spawn para rodar o loop em paralelo
                while rainbowActive do
                    local startTime = tick()
                    local endTime = startTime + transitionTime

                    local startColor = randomColor()
                    local endColor = randomColor()

                    while tick() < endTime and rainbowActive do
                        local elapsedTime = tick() - startTime
                        local alpha = elapsedTime / transitionTime

                        for _, part in ipairs(character:GetDescendants()) do
                            if part:IsA("BasePart") then
                                part.Color = lerpColor(startColor, endColor, alpha)
                                part.Transparency = 0.35
                                part.Material = Enum.Material.ForceField -- Material ForceField
                            end
                        end

                        wait()
                    end
                end
            end)
        else
            rainbowActive = false

            -- Caso o toggle seja desligado, restaurar a aparência original
            for _, part in ipairs(character:GetDescendants()) do
                if part:IsA("BasePart") then
                    part.Color = Color3.fromRGB(255, 255, 255)  -- Deixa a cor normal (branco)
                    part.Transparency = 0
                    part.Material = Enum.Material.SmoothPlastic  -- Material padrão
                end
            end
        end
    end
})

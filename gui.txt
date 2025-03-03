local Players = game:GetService("Players")
local player = Players.LocalPlayer

-- Criando a interface
local screenGui = Instance.new("ScreenGui")
screenGui.Parent = player:WaitForChild("PlayerGui")

local frame = Instance.new("Frame")
frame.Size = UDim2.new(0.2, 0, 0.3, 0)  -- Ajuste dinâmico de tamanho
frame.Position = UDim2.new(0.8, 0, 0.3, 0)
frame.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
frame.BackgroundTransparency = 0.2  -- Tornando o fundo transparente
frame.BorderSizePixel = 0  -- Removendo borda
frame.Radius = UDim.new(0, 15)  -- Bordas arredondadas
frame.ClipsDescendants = true
frame.Parent = screenGui

-- Criando um título
local title = Instance.new("TextLabel")
title.Size = UDim2.new(1, 0, 0, 40)
title.BackgroundTransparency = 1
title.Text = "Yuri Menu"
title.TextSize = 18
title.TextColor3 = Color3.fromRGB(255, 255, 255)
title.TextFont = Enum.Font.Gotham
title.TextAlignment = Enum.TextAlignment.Center
title.Parent = frame

-- Função para criar botões
local function createButton(name, position, callback)
    local button = Instance.new("TextButton")
    button.Size = UDim2.new(1, -10, 0, 40)
    button.Position = UDim2.new(0, 5, 0, position)
    button.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
    button.TextColor3 = Color3.fromRGB(255, 255, 255)
    button.TextSize = 16
    button.Text = name
    button.TextFont = Enum.Font.Gotham
    button.TextAlignment = Enum.TextAlignment.Center
    button.Parent = frame
    
    -- Efeitos de interação com o botão
    button.MouseEnter:Connect(function()
        button.BackgroundColor3 = Color3.fromRGB(60, 60, 60)  -- Cor ao passar o mouse
    end)

    button.MouseLeave:Connect(function()
        button.BackgroundColor3 = Color3.fromRGB(50, 50, 50)  -- Cor original
    end)

    button.MouseButton1Click:Connect(callback)
end

-- Criando os botões com espaços para os scripts
createButton("🔒 Aim Lock", 50, function()
    print("Aim Lock ativado!") 
    local Players = game:GetService("Players")
    local RunService = game:GetService("RunService")
    local UserInputService = game:GetService("UserInputService")
    local localPlayer = Players.LocalPlayer
    local camera = workspace.CurrentCamera

    local aimLockedPlayer = nil -- Alvo do Aim Lock
    local isAimingLocked = false -- Controle do estado

    -- Distância máxima para travar a mira
    local AIMLOCK_DISTANCE = 500 -- Exploits normalmente ignoram limites, então deixamos alto

    -- Encontra o jogador mais próximo, ignorando paredes
    local function getClosestPlayer()
        local closestPlayer = nil
        local shortestDistance = AIMLOCK_DISTANCE
        
        for _, player in pairs(Players:GetPlayers()) do
            if player ~= localPlayer and player.Character and player.Character:FindFirstChild("Head") then
                local head = player.Character.Head
                local distance = (head.Position - localPlayer.Character.Head.Position).Magnitude

                if distance < shortestDistance then
                    closestPlayer = player
                    shortestDistance = distance
                end
            end
        end
        
        return closestPlayer
    end

    -- Alterna o Aim Lock ao pressionar "Q"
    UserInputService.InputBegan:Connect(function(input, gameProcessed)
        if input.KeyCode == Enum.KeyCode.Q then
            if isAimingLocked then
                aimLockedPlayer = nil
                isAimingLocked = false
                print("🔓 Aim Lock desativado.")
            else
                local target = getClosestPlayer()
                if target then
                    aimLockedPlayer = target
                    isAimingLocked = true
                    print("🔒 Aim Lock ativado em: " .. target.Name)
                end
            end
        end
    end)

    -- Mantém travada a mira
    RunService.RenderStepped:Connect(function()
        if isAimingLocked and aimLockedPlayer and aimLockedPlayer.Character and aimLockedPlayer.Character:FindFirstChild("Head") then
            local headPosition = aimLockedPlayer.Character.Head.Position
            camera.CFrame = CFrame.new(camera.CFrame.Position, headPosition) -- Mira diretamente no alvo
        end
    end)

end)

createButton("👀 ESP", 100, function()
    print("ESP ativado!") 
    local Players = game:GetService("Players")

    -- Função para o ESP em cima do player
    local function createESP(player)
        if player.Character and player.Character:FindFirstChild("Head") then
            local head = player.Character.Head
            local billboard = Instance.new("BillboardGui")
            billboard.Adornee = head
            billboard.Size = UDim2.new(2, 0, 1, 0)
            billboard.StudsOffset = Vector3.new(0, 2, 0)
            billboard.AlwaysOnTop = true

            local textLabel = Instance.new("TextLabel", billboard)
            textLabel.Size = UDim2.new(1, 0, 1, 0)
            textLabel.BackgroundTransparency = 1
            textLabel.Text = player.Name
            textLabel.TextColor3 = Color3.fromRGB(255, 0, 0) -- Vermelho
            textLabel.TextStrokeTransparency = 0.5
            textLabel.Font = Enum.Font.SourceSansBold
            textLabel.TextScaled = true

            billboard.Parent = head
        end
    end

    -- Adiciona ESP em jogadores já presentes
    for _, player in pairs(Players:GetPlayers()) do
        if player ~= Players.LocalPlayer then
            createESP(player)
        end
    end

    -- Adiciona ESP em jogadores que entram
    Players.PlayerAdded:Connect(function(player)
        player.CharacterAdded:Connect(function()
            wait(1) -- Espera o personagem carregar
            createESP(player)
        end)
    end)
end)

createButton("🎯 Aim Assist", 150, function()
    print("Aim Assist ativado!") 
    local Players = game:GetService("Players")
    local RunService = game:GetService("RunService")
    local UserInputService = game:GetService("UserInputService")

    local localPlayer = Players.LocalPlayer
    local camera = workspace.CurrentCamera
    local aiming = false

    -- Ativa ao segurar botão direito do mouse
    UserInputService.InputBegan:Connect(function(input, gameProcessed)
        if input.UserInputType == Enum.UserInputType.MouseButton2 then
            aiming = true
        end
    end)

    UserInputService.InputEnded:Connect(function(input, gameProcessed)
        if input.UserInputType == Enum.UserInputType.MouseButton2 then
            aiming = false
        end
    end)

    -- Função para encontrar o player mais próximo
    local function getClosestPlayer()
        local closestPlayer = nil
        local shortestDistance = math.huge
        local mouseLocation = UserInputService:GetMouseLocation()

        for _, player in pairs(Players:GetPlayers()) do
            if player ~= localPlayer and player.Character and player.Character:FindFirstChild("Head") then
                local headPosition, onScreen = camera:WorldToViewportPoint(player.Character.Head.Position)

                if onScreen then
                    local distance = (Vector2.new(headPosition.X, headPosition.Y) - mouseLocation).Magnitude

                    if distance < shortestDistance and distance < 100 then -- Máximo de 100 pixels de distância para travar
                        closestPlayer = player
                        shortestDistance = distance
                    end
                end
            end
        end

        return closestPlayer
    end

    -- Atualiza a câmera para travar na mira
    RunService.RenderStepped:Connect(function()
        if aiming then
            local targetPlayer = getClosestPlayer()
            
            if targetPlayer and targetPlayer.Character and targetPlayer.Character:FindFirstChild("Head") then
                local headPosition = targetPlayer.Character.Head.Position
                local direction = (headPosition - camera.CFrame.Position).unit
                local newCFrame = CFrame.new(camera.CFrame.Position, camera.CFrame.Position + direction)

                camera.CFrame = camera.CFrame:Lerp(newCFrame, 0.2) -- Suaviza o movimento da mira
            end
        end
    end)
end)

-- Função para abrir/fechar a interface com a tecla Insert
local UserInputService = game:GetService("UserInputService")
local isMenuOpen = true  -- Inicializa a interface como visível

UserInputService.InputBegan:Connect(function(input, gameProcessed)
    if input.KeyCode == Enum.KeyCode.Insert then
        isMenuOpen = not isMenuOpen  -- Alterna o estado da interface
        frame.Visible = isMenuOpen  -- Torna o frame visível ou invisível
    end
end)

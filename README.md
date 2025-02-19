-- Configurações
local Settings = {
    Box_Color = Color3.fromRGB(255, 0, 0),
    Box_Thickness = 2,
    Team_Check = false,
    Team_Color = false,
    Autothickness = true
}

-- Variáveis locais
local Space = game:GetService("Workspace")
local Player = game:GetService("Players").LocalPlayer
local Camera = Space.CurrentCamera
local UserInputService = game:GetService("UserInputService")

-- Variável de controle do ESP (ativado ou desativado)
local ESP_Active = true

-- Função para desenhar as linhas
local function NewLine(color, thickness)
    local line = Drawing.new("Line")
    line.Visible = false
    line.From = Vector2.new(0, 0)
    line.To = Vector2.new(0, 0)
    line.Color = color
    line.Thickness = thickness
    line.Transparency = 1
    return line
end

-- Função para controlar a visibilidade das linhas
local function Vis(lib, state)
    for i, v in pairs(lib) do
        v.Visible = state
    end
end

-- Função para alterar as cores das linhas
local function Colorize(lib, color)
    for i, v in pairs(lib) do
        v.Color = color
    end
end

-- Função para a animação de cores arco-íris
local function Rainbow(lib, delay)
    for hue = 0, 1, 1/30 do
        local color = Color3.fromHSV(hue, 0.6, 1)
        Colorize(lib, color)
        wait(delay)
    end
    Rainbow(lib)
end

-- Função principal do ESP
local function Main(plr)
    repeat wait() until plr.Character ~= nil and plr.Character:FindFirstChild("Humanoid") ~= nil
    local R15
    if plr.Character.Humanoid.RigType == Enum.HumanoidRigType.R15 then
        R15 = true
    else 
        R15 = false
    end
    local Library = {
        TL1 = NewLine(Settings.Box_Color, Settings.Box_Thickness),
        TL2 = NewLine(Settings.Box_Color, Settings.Box_Thickness),
        TR1 = NewLine(Settings.Box_Color, Settings.Box_Thickness),
        TR2 = NewLine(Settings.Box_Color, Settings.Box_Thickness),
        BL1 = NewLine(Settings.Box_Color, Settings.Box_Thickness),
        BL2 = NewLine(Settings.Box_Color, Settings.Box_Thickness),
        BR1 = NewLine(Settings.Box_Color, Settings.Box_Thickness),
        BR2 = NewLine(Settings.Box_Color, Settings.Box_Thickness)
    }
    coroutine.wrap(Rainbow)(Library, 0.15)
    local oripart = Instance.new("Part")
    oripart.Parent = Space
    oripart.Transparency = 1
    oripart.CanCollide = false
    oripart.Size = Vector3.new(1, 1, 1)
    oripart.Position = Vector3.new(0, 0, 0)

    -- Função para atualizar o ESP
    local function Updater()
        local c 
        c = game:GetService("RunService").RenderStepped:Connect(function()
            if ESP_Active then
                if plr.Character ~= nil and plr.Character:FindFirstChild("Humanoid") ~= nil and plr.Character:FindFirstChild("HumanoidRootPart") ~= nil and plr.Character.Humanoid.Health > 0 and plr.Character:FindFirstChild("Head") ~= nil then
                    local Hum = plr.Character
                    local HumPos, vis = Camera:WorldToViewportPoint(Hum.HumanoidRootPart.Position)
                    if vis then
                        oripart.Size = Vector3.new(Hum.HumanoidRootPart.Size.X, Hum.HumanoidRootPart.Size.Y*1.5, Hum.HumanoidRootPart.Size.Z)
                        oripart.CFrame = CFrame.new(Hum.HumanoidRootPart.CFrame.Position, Camera.CFrame.Position)
                        local SizeX = oripart.Size.X
                        local SizeY = oripart.Size.Y
                        local TL = Camera:WorldToViewportPoint((oripart.CFrame * CFrame.new(SizeX, SizeY, 0)).p)
                        local TR = Camera:WorldToViewportPoint((oripart.CFrame * CFrame.new(-SizeX, SizeY, 0)).p)
                        local BL = Camera:WorldToViewportPoint((oripart.CFrame * CFrame.new(SizeX, -SizeY, 0)).p)
                        local BR = Camera:WorldToViewportPoint((oripart.CFrame * CFrame.new(-SizeX, -SizeY, 0)).p)

                        if Settings.Team_Check then
                            if plr.TeamColor == Player.TeamColor then
                                Colorize(Library, Color3.fromRGB(0, 255, 0))
                            else 
                                Colorize(Library, Color3.fromRGB(255, 0, 0))
                            end
                        end

                        if Settings.Team_Color then
                            Colorize(Library, plr.TeamColor.Color)
                        end

                        local ratio = (Camera.CFrame.p - Hum.HumanoidRootPart.Position).magnitude
                        local offset = math.clamp(1/ratio*750, 2, 300)

                        Library.TL1.From = Vector2.new(TL.X, TL.Y)
                        Library.TL1.To = Vector2.new(TL.X + offset, TL.Y)
                        Library.TL2.From = Vector2.new(TL.X, TL.Y)
                        Library.TL2.To = Vector2.new(TL.X, TL.Y + offset)

                        Library.TR1.From = Vector2.new(TR.X, TR.Y)
                        Library.TR1.To = Vector2.new(TR.X - offset, TR.Y)
                        Library.TR2.From = Vector2.new(TR.X, TR.Y)
                        Library.TR2.To = Vector2.new(TR.X, TR.Y + offset)

                        Library.BL1.From = Vector2.new(BL.X, BL.Y)
                        Library.BL1.To = Vector2.new(BL.X + offset, BL.Y)
                        Library.BL2.From = Vector2.new(BL.X, BL.Y)
                        Library.BL2.To = Vector2.new(BL.X, BL.Y - offset)

                        Library.BR1.From = Vector2.new(BR.X, BR.Y)
                        Library.BR1.To = Vector2.new(BR.X - offset, BR.Y)
                        Library.BR2.From = Vector2.new(BR.X, BR.Y)
                        Library.BR2.To = Vector2.new(BR.X, BR.Y - offset)

                        Vis(Library, true)

                        if Settings.Autothickness then
                            local distance = (Player.Character.HumanoidRootPart.Position - oripart.Position).magnitude
                            local value = math.clamp(1/distance*100, 1, 4)
                            for u, x in pairs(Library) do
                                x.Thickness = value
                            end
                        else 
                            for u, x in pairs(Library) do
                                x.Thickness = Settings.Box_Thickness
                            end
                        end
                    else 
                        Vis(Library, false)
                    end
                else 
                    Vis(Library, false)
                    if game:GetService("Players"):FindFirstChild(plr.Name) == nil then
                        for i, v in pairs(Library) do
                            v:Remove()
                            oripart:Destroy()
                        end
                        c:Disconnect()
                    end
                end
            else
                Vis(Library, false)
            end
        end)
    end

    -- Chama a função de atualização
    coroutine.wrap(Updater)()
end

-- Função para desativar o ESP
local function DisableESP()
    ESP_Active = false
end

-- Função para ativar o ESP
local function EnableESP()
    ESP_Active = true
end

-- Detectando a tecla "Y" para alternar o estado do ESP
UserInputService.InputBegan:Connect(function(input, gameProcessed)
    if not gameProcessed then
        if input.KeyCode == Enum.KeyCode.Y then
            if ESP_Active then
                DisableESP()
            else
                EnableESP()
            end
        end
    end
end)

-- Ativa o ESP para todos os jogadores
for i, v in pairs(game:GetService("Players"):GetPlayers()) do
    if v.Name ~= Player.Name then
        coroutine.wrap(Main)(v)
    end
end

game:GetService("Players").PlayerAdded:Connect(function(newplr)
    coroutine.wrap(Main)(newplr)
end)

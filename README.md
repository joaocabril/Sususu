local __GetService = setmetatable({}, {
    __index = function(self, key)
        assert(type(key) == "string", "Service key must be a string.")
        local __BypassedService = rawget(self, key)
        if not __BypassedService then
            local success, result = pcall(function()
                return (cloneref or function(...) return ... end)(game:GetService(key))
            end)
            if success then
                __BypassedService = result
                rawset(self, key, __BypassedService)
            else
                warn("Invalid service: " .. key)
                return nil
            end
        end
    return __BypassedService
end})

local LocalPlayer = __GetService["Players"].LocalPlayer
local Workspace = __GetService["Workspace"]
local TweenService = __GetService.TweenService

local UserInputService = __GetService.UserInputService
local RunService = __GetService.RunService
local NextFrame = RunService.Heartbeat

local OwnerName = LocalPlayer.Character.Name
local Inventario = LocalPlayer.Inventario
local Backpack = LocalPlayer.Backpack
local GetCheckPote = Workspace:FindFirstChild(OwnerName):FindFirstChild("PoteErva")
local Character = LocalPlayer.Character or LocalPlayer.CharacterAdded:Wait()

local Connections = {}

getgenv().TweenSpeed = 70

-- // Systema Para Verificar O Requirimentos
function getRoot(Client)
    local rootPart = Client:FindFirstChild("HumanoidRootPart") or Client:FindFirstChild("Torso") or Client:FindFirstChild("UpperTorso") 
   return rootPart
end

-- // Utility Table
local Utility = {}
do
    Utility.new_connection = function(type, callback)
        local connection = type:Connect(callback)
        table.insert(Connections, connection)
        return connection
    end;

    Utility.remove_connection = function(connection)
        for i, v in pairs(Connections) do
            if v == connection then
                Connections[i] = nil
                v:Disconnect()
            end
        end
    end;

    Utility.has_character = function(client)
        return (client and client.Character and client.Character:FindFirstChild("Humanoid")) and true or false
    end

    Utility.GetDistance = function(Endpoint)
        if typeof(Endpoint) == "Instance" then
            Endpoint = Vector3.new(Endpoint.Position.X, LocalPlayer.Character.HumanoidRootPart.Position.Y, Endpoint.Position.Z)
        elseif typeof(Endpoint) == "CFrame" then
            Endpoint = Vector3.new(Endpoint.Position.X, LocalPlayer.Character.HumanoidRootPart.Position.Y, Endpoint.Position.Z)
        end
        return (Endpoint - LocalPlayer.Character.HumanoidRootPart.Position).Magnitude
    end;

    Utility.CheckPos = function(Pos, MaxDistance)
        if typeof(Pos) == "Instance" then
            Pos = Pos.CFrame
        end
    
        if Utility.GetDistance(Pos) < MaxDistance then
           return true
        end
        return false
    end;
    
    Utility.Tween = function(Endpoint)
        local Character = getRoot(LocalPlayer.Character)

        if typeof(Endpoint) == "Instance" then
            Endpoint = Endpoint.CFrame
        end

        local TweenFunc = {}
        local Distance = Utility.GetDistance(Endpoint)

        if Distance <= 40 then
            LocalPlayer.Character.HumanoidRootPart.CFrame = Endpoint
            return false
        end

        local TweenInfo = TweenInfo.new(Distance / getgenv().TweenSpeed, Enum.EasingStyle.Linear)
        local Tween = __GetService.TweenService:Create(LocalPlayer.Character.HumanoidRootPart, TweenInfo, {CFrame = Endpoint})

        local BodyGyro = Instance.new("BodyGyro")
        BodyGyro.P = 9e4
        BodyGyro.maxTorque = Vector3.new(9e9, 9e9, 9e9)
        BodyGyro.CFrame = Character.CFrame
        BodyGyro.Parent = Character

        local BodyVelocity = Instance.new("BodyVelocity")
        BodyVelocity.Velocity = Vector3.new(0, 0, 0)
        BodyVelocity.MaxForce = Vector3.new(9e9, 9e9, 9e9)
        BodyVelocity.Parent = Character

        Tween:Play()

        function TweenFunc:Cancel()
            Tween:Cancel()
            return false
        end

        Tween.Completed:Wait()
        BodyGyro:Destroy()
        BodyVelocity:Destroy()
        return TweenFunc
    end;

    Utility.TweenType2 = function(Endpoint)
        local Character = getRoot(LocalPlayer.Character)
    
        if typeof(Endpoint) == "Instance" then
            Endpoint = Endpoint.CFrame
        end
    
        local TweenFunc = {}
        local Distance = Utility.GetDistance(Endpoint)
    
        if Distance <= 50 then
               LocalPlayer.Character.HumanoidRootPart.CFrame = Endpoint
            return false
        end

        local TweenInfo = TweenInfo.new(Distance / getgenv().TweenSpeed, Enum.EasingStyle.Linear)
        local Tween = TweenService:Create(LocalPlayer.Character.HumanoidRootPart, TweenInfo, {CFrame = Endpoint})
    
        local BodyGyro = Instance.new("BodyGyro")
        BodyGyro.P = 9e4
        BodyGyro.MaxTorque = Vector3.new(9e9, 9e9, 9e9)
        BodyGyro.CFrame = Character.CFrame
        BodyGyro.Parent = Character

        local BodyVelocity = Instance.new("BodyVelocity")
        BodyVelocity.Velocity = Vector3.new(0, 0, 0)
        BodyVelocity.MaxForce = Vector3.new(9e9, 9e9, 9e9)
        BodyVelocity.Parent = Character
    
        Tween:Play()
    
        Tween.Completed:Wait()

        return TweenFunc
    end;
    
    Utility.SetPromt = function(Promt)
        if not (typeof(Promt) == "Instance" and Promt:IsA("ProximityPrompt")) then
            error("Expected ProximityPrompt, got " .. (Promt.ClassName or "nil"))
        end
    
        local Configuration = {
            RequiresLineOfSight = Promt.RequiresLineOfSight,
            MaxActivationDistance = Promt.MaxActivationDistance,
            HoldDuration = Promt.HoldDuration,
            Enabled = Promt.Enabled
        }
    
        Promt.RequiresLineOfSight = false
        Promt.MaxActivationDistance = math.huge 
        Promt.HoldDuration = 0.05
        Promt.Enabled = true
    
        task.spawn(function()
            task.wait()
            Promt:InputHoldBegin()
            task.wait(0.05)
            Promt:InputHoldEnd()
    
            task.spawn(function()
                task.wait(0.01)
                Promt.Enabled = Configuration.Enabled
                Promt.MaxActivationDistance = Configuration.MaxActivationDistance
                Promt.RequiresLineOfSight = Configuration.RequiresLineOfSight
                Promt.HoldDuration = Configuration.HoldDuration
            end)
        end)
    end;
end

-- // Systema Pra Equipar Os Potes
local GetPote = function()
    local potes = {} 
    local equipado = Character:FindFirstChild("PoteErva") 
    
    for _, item in ipairs(Backpack:GetChildren()) do
        if item:IsA("Tool") and item.Name == "PoteErva" then
            table.insert(potes, item)
        end
    end
    
    if #potes > 0 then
        if not equipado then
            local poteParaEquipar = nil
            for _, pote in ipairs(potes) do
                local handle = pote:FindFirstChild("Handle")
                if handle and handle:FindFirstChild("qnts") then
                    local quantidade = handle.qnts.Value
                    if quantidade < 10 then
                        poteParaEquipar = pote
                        break
                    end
                end
            end

            if poteParaEquipar then
                local args = {[1] = "PoteErva", [2] = "Equipar"}
                game:GetService("ReplicatedStorage"):WaitForChild("InventarioSystem"):WaitForChild("Equipar"):FireServer(unpack(args))
                task.wait(0.2)
                poteParaEquipar.Parent = Character 
            end
        end

        if equipado then
            local handle = equipado:FindFirstChild("Handle")
            if handle and handle:FindFirstChild("qnts") then
                local quantidade = handle.qnts.Value

                if quantidade == 10 then
                    equipado.Parent = Backpack 
                    for _, pote in ipairs(potes) do
                        if pote ~= equipado then 
                            local handle = pote:FindFirstChild("Handle")
                            if handle and handle:FindFirstChild("qnts") then
                                local quantidade = handle.qnts.Value
                                if quantidade < 10 then
                                    local args = {[1] = "PoteErva", [2] = "Equipar"}
                                    game:GetService("ReplicatedStorage"):WaitForChild("InventarioSystem"):WaitForChild("Equipar"):FireServer(unpack(args))
                                    task.wait(0.2)
                                    pote.Parent = Character 
                                    break
                                end
                            end
                        end
                    end
                end
            end
        end
    end
end;

local GetPlantinhas = function()
    local Plantinha_Ilegal = workspace.Construcoes.Plantinha_Ilegal
    for _, plantinha in ipairs(Plantinha_Ilegal:GetChildren()) do
        if plantinha:IsA("Part") and plantinha.Name == "Plantinha" then
            local db = plantinha:FindFirstChild("db")
            if db and db.Value == false then
                return plantinha
            end
        end
    end
    return nil 
end

local GetPlantinhasFarm = GetPlantinhas()

local GetPoteValue = function(pote)
    local handle = pote and pote:FindFirstChild("Handle")
    if handle and handle:FindFirstChild("qnts") then
        return handle.qnts.Value
    end
    return 0  
end

local GetPoteState = function()
    local contador = 0
    local potesEncontrados = {} 

    for _, item in ipairs(Backpack:GetChildren()) do
        if item:IsA("Tool") and item.Name == "PoteErva" then
            contador = contador + 1
            if contador > 3 then break end

            local quantidade = GetPoteValue(item)
            table.insert(potesEncontrados, {item, quantidade})
        end
    end

    if contador < 3 then
        local getCheckPoteWorkspace = Workspace:FindFirstChild(OwnerName)
        if getCheckPoteWorkspace then
            for _, pote in ipairs(getCheckPoteWorkspace:GetChildren()) do
                if pote:IsA("Tool") and pote.Name == "PoteErva" then
                    contador = contador + 1
                    if contador > 3 then break end

                    local quantidade = GetPoteValue(pote)
                    table.insert(potesEncontrados, {pote, quantidade})
                end
            end
        end
    end

    if contador == 0 then
        print("Nenhum PoteErva encontrado no inventÃ¡rio ou no Workspace.")
    end

    return contador, potesEncontrados
end

local Comprador = CFrame.new(4320, 14, -2644)

local GetSellPots = function()
    local totalPotes, potesDetalhados = GetPoteState()     
    local todosPotesCheios = true
    
    for i, pote in ipairs(potesDetalhados) do
        if pote[2] < 10 then
            todosPotesCheios = false 
        end
    end
    
    if todosPotesCheios then
        return true 
    else
        return false 
    end
end
  
local GetPoteType2 = function()
    local potesCheios = {} 
  
    for _, item in ipairs(Backpack:GetChildren()) do
        if item:IsA("Tool") and item.Name == "PoteErva" then
            local handle = item:FindFirstChild("Handle")
            if handle and handle:FindFirstChild("qnts") then
                if handle.qnts.Value == 10 then 
                    table.insert(potesCheios, item) 
                end
            end
        end
    end
  
    if #potesCheios > 0 then
        for _, pote in ipairs(potesCheios) do
            LocalPlayer.Character.Humanoid:EquipTool(pote) 
            if Utility.CheckPos(Comprador, 35) then
                Utility.SetPromt(Workspace.Construcoes.Plantinha_Ilegal.Vendedor.PROXI.ProximityPrompt)
                task.wait(0.5) 
            end
        end
  
        if GetSellPots() == false then
            potesCheios = {} 
        end
  
    else
        print("Nenhum PoteErva cheio encontrado no inventÃ¡rio.")
    end
end

--// Start Auto Farm
getgenv().AutoFarmPlants = true

while getgenv().AutoFarmPlants and task.wait() and Utility.has_character(LocalPlayer) do
    local inventario = LocalPlayer:WaitForChild("Backpack")
    local poteDeErva = inventario:FindFirstChild("PoteErva")
    local getCheckPote = Workspace:FindFirstChild(OwnerName):FindFirstChild("PoteErva")

    if getCheckPote then
         local GetYouPoteValue = Workspace:FindFirstChild(OwnerName):FindFirstChild("PoteErva").Handle
         local Value = GetYouPoteValue.qnts.Value
       if GetSellPots() == true then
        Utility.Tween(Comprador) 
        task.wait()
        GetPoteType2()
        task.wait(3)
      elseif GetSellPots() == false then
        local GetFarms = GetPlantinhas()
           if GetFarms then
            GetPote()
            local position = GetFarms.Position
            Utility.TweenType2(CFrame.new(position.X, position.Y + 3, position.Z)) 
            Utility.SetPromt(GetFarms:WaitForChild("ProximityPrompt"))
            task.wait(0.5) 
        end
      end
    else
        GetPote()
    end
end;

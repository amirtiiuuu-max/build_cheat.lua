-- ========================================
--  DELTA BUILD STUDIO - РАБОЧАЯ ВЕРСИЯ
--  Без лишних зависимостей, всё в одном файле
-- ========================================

local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local Workspace = game:GetService("Workspace")
local LocalPlayer = Players.LocalPlayer
local Mouse = LocalPlayer:GetMouse()

-- ========== НАСТРОЙКИ ==========
local buildMode = false
local selectedPart = nil
local builtParts = {}

-- Текущие параметры блока
local currentShape = "Block"
local currentColor = Color3.fromRGB(255, 100, 50)
local currentSize = Vector3.new(2, 2, 2)
local currentMaterial = Enum.Material.Plastic

-- Списки
local shapes = {"Block", "Sphere", "Cylinder", "Wedge", "CornerWedge", "Pyramid", "Cone", "Prism"}
local materials = {"Plastic", "Wood", "Brick", "Neon", "Glass"}

-- Функция создания блока
local function CreatePart(shape, color, size, material, pos)
    local part
    if shape == "Block" then
        part = Instance.new("Part")
        part.Shape = Enum.PartType.Block
    elseif shape == "Sphere" then
        part = Instance.new("Part")
        part.Shape = Enum.PartType.Ball
    elseif shape == "Cylinder" then
        part = Instance.new("Part")
        part.Shape = Enum.PartType.Cylinder
    elseif shape == "Wedge" then
        part = Instance.new("WedgePart")
    elseif shape == "CornerWedge" then
        part = Instance.new("CornerWedgePart")
    elseif shape == "Pyramid" then
        part = Instance.new("Part")
        local m = Instance.new("SpecialMesh")
        m.MeshType = Enum.MeshType.FileMesh
        m.MeshId = "rbxassetid://242891189"
        m.Scale = size / 2
        m.Parent = part
    elseif shape == "Cone" then
        part = Instance.new("Part")
        local m = Instance.new("SpecialMesh")
        m.MeshType = Enum.MeshType.Cone
        m.Scale = Vector3.new(size.X/2, size.Y/2, size.Z/2)
        m.Parent = part
    elseif shape == "Prism" then
        part = Instance.new("Part")
        local m = Instance.new("SpecialMesh")
        m.MeshType = Enum.MeshType.FileMesh
        m.MeshId = "rbxassetid://242891191"
        m.Scale = size / 2
        m.Parent = part
    else
        part = Instance.new("Part")
    end
    part.Size = size
    part.Color = color
    part.BrickColor = BrickColor.new(color)
    part.Material = material
    part.Position = pos
    part.Anchored = true
    part.Parent = Workspace
    
    -- Подсветка выделения
    local selBox = Instance.new("SelectionBox")
    selBox.Adornee = part
    selBox.Color3 = Color3.fromRGB(0,255,255)
    selBox.LineThickness = 0.05
    selBox.Visible = false
    selBox.Parent = part
    return part
end

-- Выделение блока
local function SelectPart(part)
    if selectedPart then
        local box = selectedPart:FindFirstChildWhichIsA("SelectionBox")
        if box then box.Visible = false end
    end
    selectedPart = part
    if selectedPart then
        local box = selectedPart:FindFirstChildWhichIsA("SelectionBox")
        if box then box.Visible = true end
    end
end

-- Расширение выделенного блока
local function ExpandSelected(direction, amount)
    if not selectedPart then return end
    local size = selectedPart.Size
    local pos = selectedPart.Position
    if direction == "X+" then
        size = size + Vector3.new(amount, 0, 0)
        pos = pos + Vector3.new(amount/2, 0, 0)
    elseif direction == "X-" then
        size = size + Vector3.new(amount, 0, 0)
        pos = pos - Vector3.new(amount/2, 0, 0)
    elseif direction == "Y+" then
        size = size + Vector3.new(0, amount, 0)
        pos = pos + Vector3.new(0, amount/2, 0)
    elseif direction == "Y-" then
        size = size + Vector3.new(0, amount, 0)
        pos = pos - Vector3.new(0, amount/2, 0)
    elseif direction == "Z+" then
        size = size + Vector3.new(0, 0, amount)
        pos = pos + Vector3.new(0, 0, amount/2)
    elseif direction == "Z-" then
        size = size + Vector3.new(0, 0, amount)
        pos = pos - Vector3.new(0, 0, amount/2)
    end
    selectedPart.Size = size
    selectedPart.Position = pos
    -- Обновить меш для пирамид/конусов
    local mesh = selectedPart:FindFirstChildWhichIsA("SpecialMesh")
    if mesh then
        if mesh.MeshType == Enum.MeshType.Cone then
            mesh.Scale = Vector3.new(size.X/2, size.Y/2, size.Z/2)
        elseif mesh.MeshType == Enum.MeshType.FileMesh then
            mesh.Scale = size / 2
        end
    end
end

-- ========== GUI ==========
local screenGui = Instance.new("ScreenGui")
screenGui.Name = "BuildGUI"
screenGui.Parent = game:GetService("CoreGui")

local mainFrame = Instance.new("Frame")
mainFrame.Size = UDim2.new(0, 350, 0, 500)
mainFrame.Position = UDim2.new(0, 10, 0, 10)
mainFrame.BackgroundColor3 = Color3.fromRGB(30,30,40)
mainFrame.BackgroundTransparency = 0.1
mainFrame.BorderSizePixel = 0
mainFrame.Parent = screenGui
local corner = Instance.new("UICorner")
corner.CornerRadius = UDim.new(0, 8)
corner.Parent = mainFrame

local title = Instance.new("TextLabel")
title.Size = UDim2.new(1,0,0,35)
title.BackgroundColor3 = Color3.fromRGB(45,45,55)
title.Text = "DELTA BUILD STUDIO"
title.TextColor3 = Color3.fromRGB(255,255,255)
title.TextScaled = true
title.Font = Enum.Font.GothamBold
title.Parent = mainFrame

-- Drag
local drag = false
local dragStart, startPos
title.InputBegan:Connect(function(i)
    if i.UserInputType == Enum.UserInputType.MouseButton1 then
        drag = true
        dragStart = i.Position
        startPos = mainFrame.Position
    end
end)
UserInputService.InputEnded:Connect(function(i)
    if i.UserInputType == Enum.UserInputType.MouseButton1 then drag = false end
end)
UserInputService.InputChanged:Connect(function(i)
    if drag and i.UserInputType == Enum.UserInputType.MouseMovement then
        local delta = i.Position - dragStart
        mainFrame.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
    end
end)

-- Close
local close = Instance.new("TextButton")
close.Size = UDim2.new(0,30,0,30)
close.Position = UDim2.new(1,-35,0,2)
close.BackgroundColor3 = Color3.fromRGB(200,60,60)
close.Text = "X"
close.TextColor3 = Color3.fromRGB(255,255,255)
close.TextScaled = true
close.Font = Enum.Font.GothamBold
close.Parent = mainFrame
close.MouseButton1Click:Connect(function() screenGui:Destroy() end)

-- Контейнер с прокруткой
local scroll = Instance.new("ScrollingFrame")
scroll.Size = UDim2.new(1,-20,1,-50)
scroll.Position = UDim2.new(0,10,0,45)
scroll.BackgroundTransparency = 1
scroll.CanvasSize = UDim2.new(0,0,0,0)
scroll.ScrollBarThickness = 6
scroll.Parent = mainFrame
local layout = Instance.new("UIListLayout")
layout.Padding = UDim.new(0,8)
layout.Parent = scroll

-- Кнопка Build Mode
local buildBtn = Instance.new("TextButton")
buildBtn.Size = UDim2.new(1,0,0,40)
buildBtn.BackgroundColor3 = Color3.fromRGB(60,60,75)
buildBtn.Text = "🔨 BUILD MODE: OFF"
buildBtn.TextColor3 = Color3.fromRGB(255,255,255)
buildBtn.TextScaled = true
buildBtn.Font = Enum.Font.GothamBold
buildBtn.Parent = scroll
buildBtn.MouseButton1Click:Connect(function()
    buildMode = not buildMode
    buildBtn.Text = buildMode and "🔨 BUILD MODE: ON" or "🔨 BUILD MODE: OFF"
    buildBtn.BackgroundColor3 = buildMode and Color3.fromRGB(60,120,60) or Color3.fromRGB(60,60,75)
end)

-- Форма
local shapeBtn = Instance.new("TextButton")
shapeBtn.Size = UDim2.new(1,0,0,35)
shapeBtn.BackgroundColor3 = Color3.fromRGB(50,50,65)
shapeBtn.Text = "Форма: " .. currentShape
shapeBtn.TextColor3 = Color3.fromRGB(255,255,255)
shapeBtn.TextScaled = true
shapeBtn.Font = Enum.Font.Gotham
shapeBtn.Parent = scroll
local shapeList = nil
shapeBtn.MouseButton1Click:Connect(function()
    if shapeList then shapeList:Destroy() shapeList = nil return end
    shapeList = Instance.new("Frame")
    shapeList.Size = UDim2.new(1,0,0, #shapes * 30)
    shapeList.BackgroundColor3 = Color3.fromRGB(40,40,55)
    shapeList.Parent = scroll
    for i,s in ipairs(shapes) do
        local b = Instance.new("TextButton")
        b.Size = UDim2.new(1,0,0,30)
        b.BackgroundTransparency = 0.3
        b.Text = s
        b.TextColor3 = Color3.fromRGB(255,255,255)
        b.TextScaled = true
        b.Font = Enum.Font.Gotham
        b.Parent = shapeList
        b.MouseButton1Click:Connect(function()
            currentShape = s
            shapeBtn.Text = "Форма: " .. s
            shapeList:Destroy()
            shapeList = nil
        end)
    end
end)

-- Материал
local matBtn = Instance.new("TextButton")
matBtn.Size = UDim2.new(1,0,0,35)
matBtn.BackgroundColor3 = Color3.fromRGB(50,50,65)
matBtn.Text = "Материал: Plastic"
matBtn.TextColor3 = Color3.fromRGB(255,255,255)
matBtn.TextScaled = true
matBtn.Font = Enum.Font.Gotham
matBtn.Parent = scroll
local matList = nil
matBtn.MouseButton1Click:Connect(function()
    if matList then matList:Destroy() matList = nil return end
    matList = Instance.new("Frame")
    matList.Size = UDim2.new(1,0,0, #materials * 30)
    matList.BackgroundColor3 = Color3.fromRGB(40,40,55)
    matList.Parent = scroll
    for i,m in ipairs(materials) do
        local b = Instance.new("TextButton")
        b.Size = UDim2.new(1,0,0,30)
        b.BackgroundTransparency = 0.3
        b.Text = m
        b.TextColor3 = Color3.fromRGB(255,255,255)
        b.TextScaled = true
        b.Font = Enum.Font.Gotham
        b.Parent = matList
        b.MouseButton1Click:Connect(function()
            currentMaterial = Enum.Material[m]
            matBtn.Text = "Материал: " .. m
            matList:Destroy()
            matList = nil
        end)
    end
end)

-- Размер X
local frameX = Instance.new("Frame")
frameX.Size = UDim2.new(1,0,0,40)
frameX.BackgroundTransparency = 1
frameX.Parent = scroll
local xLabel = Instance.new("TextLabel")
xLabel.Size = UDim2.new(0.2,0,1,0)
xLabel.Text = "X"
xLabel.TextColor3 = Color3.fromRGB(255,100,100)
xLabel.TextScaled = true
xLabel.Font = Enum.Font.GothamBold
xLabel.Parent = frameX
local xMinus = Instance.new("TextButton")
xMinus.Size = UDim2.new(0.25,0,1,0)
xMinus.Position = UDim2.new(0.25,0,0,0)
xMinus.Text = "-0.5"
xMinus.BackgroundColor3 = Color3.fromRGB(60,60,80)
xMinus.TextColor3 = Color3.fromRGB(255,255,255)
xMinus.TextScaled = true
xMinus.Parent = frameX
local xVal = Instance.new("TextBox")
xVal.Size = UDim2.new(0.25,0,1,0)
xVal.Position = UDim2.new(0.5,0,0,0)
xVal.BackgroundColor3 = Color3.fromRGB(50,50,70)
xVal.Text = tostring(currentSize.X)
xVal.TextColor3 = Color3.fromRGB(255,255,255)
xVal.TextScaled = true
xVal.Parent = frameX
local xPlus = Instance.new("TextButton")
xPlus.Size = UDim2.new(0.25,0,1,0)
xPlus.Position = UDim2.new(0.75,0,0,0)
xPlus.Text = "+0.5"
xPlus.BackgroundColor3 = Color3.fromRGB(60,60,80)
xPlus.TextColor3 = Color3.fromRGB(255,255,255)
xPlus.TextScaled = true
xPlus.Parent = frameX

-- Размер Y
local frameY = Instance.new("Frame")
frameY.Size = UDim2.new(1,0,0,40)
frameY.BackgroundTransparency = 1
frameY.Parent = scroll
local yLabel = Instance.new("TextLabel")
yLabel.Size = UDim2.new(0.2,0,1,0)
yLabel.Text = "Y"
yLabel.TextColor3 = Color3.fromRGB(100,255,100)
yLabel.TextScaled = true
yLabel.Font = Enum.Font.GothamBold
yLabel.Parent = frameY
local yMinus = Instance.new("TextButton")
yMinus.Size = UDim2.new(0.25,0,1,0)
yMinus.Position = UDim2.new(0.25,0,0,0)
yMinus.Text = "-0.5"
yMinus.BackgroundColor3 = Color3.fromRGB(60,60,80)
yMinus.TextColor3 = Color3.fromRGB(255,255,255)
yMinus.TextScaled = true
yMinus.Parent = frameY
local yVal = Instance.new("TextBox")
yVal.Size = UDim2.new(0.25,0,1,0)
yVal.Position = UDim2.new(0.5,0,0,0)
yVal.BackgroundColor3 = Color3.fromRGB(50,50,70)
yVal.Text = tostring(currentSize.Y)
yVal.TextColor3 = Color3.fromRGB(255,255,255)
yVal.TextScaled = true
yVal.Parent = frameY
local yPlus = Instance.new("TextButton")
yPlus.Size = UDim2.new(0.25,0,1,0)
yPlus.Position = UDim2.new(0.75,0,0,0)
yPlus.Text = "+0.5"
yPlus.BackgroundColor3 = Color3.fromRGB(60,60,80)
yPlus.TextColor3 = Color3.fromRGB(255,255,255)
yPlus.TextScaled = true
yPlus.Parent = frameY

-- Размер Z
local frameZ = Instance.new("Frame")
frameZ.Size = UDim2.new(1,0,0,40)
frameZ.BackgroundTransparency = 1
frameZ.Parent = scroll
local zLabel = Instance.new("TextLabel")
zLabel.Size = UDim2.new(0.2,0,1,0)
zLabel.Text = "Z"
zLabel.TextColor3 = Color3.fromRGB(100,100,255)
zLabel.TextScaled = true
zLabel.Font = Enum.Font.GothamBold
zLabel.Parent = frameZ
local zMinus = Instance.new("TextButton")
zMinus.Size = UDim2.new(0.25,0,1,0)
zMinus.Position = UDim2.new(0.25,0,0,0)
zMinus.Text = "-0.5"
zMinus.BackgroundColor3 = Color3.fromRGB(60,60,80)
zMinus.TextColor3 = Color3.fromRGB(255,255,255)
zMinus.TextScaled = true
zMinus.Parent = frameZ
local zVal = Instance.new("TextBox")
zVal.Size = UDim2.new(0.25,0,1,0)
zVal.Position = UDim2.new(0.5,0,0,0)
zVal.BackgroundColor3 = Color3.fromRGB(50,50,70)
zVal.Text = tostring(currentSize.Z)
zVal.TextColor3 = Color3.fromRGB(255,255,255)
zVal.TextScaled = true
zVal.Parent = frameZ
local zPlus = Instance.new("TextButton")
zPlus.Size = UDim2.new(0.25,0,1,0)
zPlus.Position = UDim2.new(0.75,0,0,0)
zPlus.Text = "+0.5"
zPlus.BackgroundColor3 = Color3.fromRGB(60,60,80)
zPlus.TextColor3 = Color3.fromRGB(255,255,255)
zPlus.TextScaled = true
zPlus.Parent = frameZ

-- Функция обновления размера
local function updateSize()
    currentSize = Vector3.new(tonumber(xVal.Text) or 1, tonumber(yVal.Text) or 1, tonumber(zVal.Text) or 1)
    xVal.Text = string.format("%.2f", currentSize.X)
    yVal.Text = string.format("%.2f", currentSize.Y)
    zVal.Text = string.format("%.2f", currentSize.Z)
end
xMinus.MouseButton1Click:Connect(function() currentSize = currentSize - Vector3.new(0.5,0,0); updateSize() end)
xPlus.MouseButton1Click:Connect(function() currentSize = currentSize + Vector3.new(0.5,0,0); updateSize() end)
yMinus.MouseButton1Click:Connect(function() currentSize = currentSize - Vector3.new(0,0.5,0); updateSize() end)
yPlus.MouseButton1Click:Connect(function() currentSize = currentSize + Vector3.new(0,0.5,0); updateSize() end)
zMinus.MouseButton1Click:Connect(function() currentSize = currentSize - Vector3.new(0,0,0.5); updateSize() end)
zPlus.MouseButton1Click:Connect(function() currentSize = currentSize + Vector3.new(0,0,0.5); updateSize() end)
xVal.FocusLost:Connect(updateSize)
yVal.FocusLost:Connect(updateSize)
zVal.FocusLost:Connect(updateSize)

-- Цвет (упрощённые пресеты)
local colorFrame = Instance.new("Frame")
colorFrame.Size = UDim2.new(1,0,0,50)
colorFrame.BackgroundTransparency = 1
colorFrame.Parent = scroll
local colorLabel = Instance.new("TextLabel")
colorLabel.Size = UDim2.new(1,0,0,20)
colorLabel.Text = "ЦВЕТ (кликни на квадрат)"
colorLabel.TextColor3 = Color3.fromRGB(200,200,200)
colorLabel.TextScaled = true
colorLabel.Font = Enum.Font.Gotham
colorLabel.Parent = colorFrame
local colorPalette = Instance.new("Frame")
colorPalette.Size = UDim2.new(1,0,0,30)
colorPalette.Position = UDim2.new(0,0,0,20)
colorPalette.BackgroundTransparency = 1
colorPalette.Parent = colorFrame
local colors = {
    Color3.fromRGB(255,0,0), Color3.fromRGB(0,255,0), Color3.fromRGB(0,0,255),
    Color3.fromRGB(255,255,0), Color3.fromRGB(255,0,255), Color3.fromRGB(0,255,255),
    Color3.fromRGB(255,255,255), Color3.fromRGB(128,128,128), Color3.fromRGB(0,0,0)
}
local xoff = 0
for i,col in ipairs(colors) do
    local btn = Instance.new("TextButton")
    btn.Size = UDim2.new(0, 35, 0, 30)
    btn.Position = UDim2.new(0, xoff, 0, 0)
    btn.BackgroundColor3 = col
    btn.Text = ""
    btn.Parent = colorPalette
    btn.MouseButton1Click:Connect(function()
        currentColor = col
    end)
    xoff = xoff + 38
    if xoff > 330 then break end
end

-- Управление выделенным блоком
local toolsTitle = Instance.new("TextLabel")
toolsTitle.Size = UDim2.new(1,0,0,25)
toolsTitle.BackgroundTransparency = 1
toolsTitle.Text = "ВЫДЕЛЕННЫЙ БЛОК (клавиша M):"
toolsTitle.TextColor3 = Color3.fromRGB(200,200,200)
toolsTitle.TextXAlignment = Enum.TextXAlignment.Left
toolsTitle.TextScaled = true
toolsTitle.Font = Enum.Font.GothamBold
toolsTitle.Parent = scroll

local expandBtns = {}
local dirs = {"X+","X-","Y+","Y-","Z+","Z-"}
for i,d in ipairs(dirs) do
    local btn = Instance.new("TextButton")
    btn.Size = UDim2.new(0.15,0,0,35)
    btn.Position = UDim2.new((i-1)*0.17,0,0,0)
    btn.BackgroundColor3 = Color3.fromRGB(70,70,90)
    btn.Text = d
    btn.TextColor3 = Color3.fromRGB(255,255,255)
    btn.TextScaled = true
    btn.Font = Enum.Font.GothamBold
    btn.Parent = scroll
    btn.MouseButton1Click:Connect(function()
        ExpandSelected(d, 0.5)
    end)
    btn.MouseButton2Click:Connect(function()
        ExpandSelected(d, -0.5)
    end)
end

local deleteBtn = Instance.new("TextButton")
deleteBtn.Size = UDim2.new(0.48,0,0,40)
deleteBtn.Position = UDim2.new(0.02,0,0,0)
deleteBtn.BackgroundColor3 = Color3.fromRGB(180,60,60)
deleteBtn.Text = "УДАЛИТЬ ВЫДЕЛЕННЫЙ"
deleteBtn.TextColor3 = Color3.fromRGB(255,255,255)
deleteBtn.TextScaled = true
deleteBtn.Font = Enum.Font.GothamBold
deleteBtn.Parent = scroll
deleteBtn.MouseButton1Click:Connect(function()
    if selectedPart then selectedPart:Destroy() end
    selectedPart = nil
end)

local deleteAllBtn = Instance.new("TextButton")
deleteAllBtn.Size = UDim2.new(0.48,0,0,40)
deleteAllBtn.Position = UDim2.new(0.5,0,0,0)
deleteAllBtn.BackgroundColor3 = Color3.fromRGB(200,40,40)
deleteAllBtn.Text = "УДАЛИТЬ ВСЕ БЛОКИ"
deleteAllBtn.TextColor3 = Color3.fromRGB(255,255,255)
deleteAllBtn.TextScaled = true
deleteAllBtn.Font = Enum.Font.GothamBold
deleteAllBtn.Parent = scroll
deleteAllBtn.MouseButton1Click:Connect(function()
    for _,p in ipairs(builtParts) do if p and p.Parent then p:Destroy() end end
    builtParts = {}
    selectedPart = nil
end)

-- Логика постройки
Mouse.Button1Down:Connect(function()
    if buildMode then
        local pos = Mouse.Hit.Position
        local newPart = CreatePart(currentShape, currentColor, currentSize, currentMaterial, pos)
        table.insert(builtParts, newPart)
        SelectPart(newPart)
    end
end)

-- Режим выделения M
UserInputService.InputBegan:Connect(function(input)
    if input.KeyCode == Enum.KeyCode.M then
        buildMode = false
        buildBtn.Text = "🔨 BUILD MODE: OFF"
        buildBtn.BackgroundColor3 = Color3.fromRGB(60,60,75)
        -- включаем режим выделения: следующий клик по блоку выделит его
        local connection
        connection = Mouse.Button1Down:Connect(function()
            local hit = Mouse.Target
            if hit and hit:IsA("BasePart") and hit.Parent == Workspace then
                SelectPart(hit)
            end
            connection:Disconnect()
        end)
        task.wait(3)
        if connection then connection:Disconnect() end
    end
end)

print("✅ СКРИПТ ЗАГРУЖЕН. ВКЛЮЧИ BUILD MODE И СТРОЙ.")

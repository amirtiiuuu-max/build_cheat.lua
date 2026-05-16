-- BUILDING CHEAT FOR DELTA (ROBLOX)
-- Features: build blocks, change shape, color, material

local Players = game:GetService("Players")
local UserInputService = game:GetService("UserInputService")
local LocalPlayer = Players.LocalPlayer
local Mouse = LocalPlayer:GetMouse()
local Workspace = game:GetService("Workspace")

-- Settings
local currentColor = Color3.fromRGB(255, 255, 255)
local currentMaterial = Enum.Material.Plastic
local currentShape = "Block" -- Block, Sphere, Cylinder, Wedge
local buildMode = false
local selectedPart = nil

-- Materials list
local materials = {
    "Plastic", "Wood", "Slate", "Concrete", "Brick", "Neon", "Glass", "Metal", "SmoothPlastic"
}

-- Shapes
local shapes = {"Block", "Sphere", "Cylinder", "Wedge"}

-- Create part function
local function CreatePart(position)
    local part
    if currentShape == "Block" then
        part = Instance.new("Part")
        part.Size = Vector3.new(1, 1, 1)
    elseif currentShape == "Sphere" then
        part = Instance.new("Part")
        part.Size = Vector3.new(1, 1, 1)
        part.Shape = Enum.PartType.Ball
    elseif currentShape == "Cylinder" then
        part = Instance.new("Part")
        part.Size = Vector3.new(1, 1, 1)
        part.Shape = Enum.PartType.Cylinder
    elseif currentShape == "Wedge" then
        part = Instance.new("WedgePart")
        part.Size = Vector3.new(1, 1, 1)
    end
    
    part.BrickColor = BrickColor.new(currentColor)
    part.Color = currentColor
    part.Material = currentMaterial
    part.Position = position
    part.Anchored = true
    part.Parent = Workspace
    return part
end

-- GUI
local screenGui = Instance.new("ScreenGui")
screenGui.Name = "BuildCheat"
screenGui.Parent = game:GetService("CoreGui")

local mainFrame = Instance.new("Frame")
mainFrame.Size = UDim2.new(0, 250, 0, 300)
mainFrame.Position = UDim2.new(0, 10, 0, 10)
mainFrame.BackgroundColor3 = Color3.fromRGB(30, 30, 40)
mainFrame.BackgroundTransparency = 0.1
mainFrame.BorderSizePixel = 0
mainFrame.Parent = screenGui

local corner = Instance.new("UICorner")
corner.CornerRadius = UDim.new(0, 8)
corner.Parent = mainFrame

-- Title
local title = Instance.new("TextLabel")
title.Size = UDim2.new(1, 0, 0, 30)
title.BackgroundColor3 = Color3.fromRGB(45, 45, 55)
title.Text = "DELTA BUILD CHEAT"
title.TextColor3 = Color3.fromRGB(255, 255, 255)
title.TextScaled = true
title.Font = Enum.Font.GothamBold
title.Parent = mainFrame

-- Build mode button
local buildBtn = Instance.new("TextButton")
buildBtn.Size = UDim2.new(1, -20, 0, 35)
buildBtn.Position = UDim2.new(0, 10, 0, 40)
buildBtn.BackgroundColor3 = Color3.fromRGB(60, 60, 70)
buildBtn.Text = "Build Mode: OFF"
buildBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
buildBtn.TextScaled = true
buildBtn.Font = Enum.Font.Gotham
buildBtn.Parent = mainFrame

local btnCorner = Instance.new("UICorner")
btnCorner.CornerRadius = UDim.new(0, 4)
btnCorner.Parent = buildBtn

-- Shape dropdown
local shapeLabel = Instance.new("TextLabel")
shapeLabel.Size = UDim2.new(0.5, -10, 0, 25)
shapeLabel.Position = UDim2.new(0, 10, 0, 85)
shapeLabel.BackgroundTransparency = 1
shapeLabel.Text = "Shape:"
shapeLabel.TextColor3 = Color3.fromRGB(220, 220, 220)
shapeLabel.TextXAlignment = Enum.TextXAlignment.Left
shapeLabel.TextScaled = true
shapeLabel.Font = Enum.Font.Gotham
shapeLabel.Parent = mainFrame

local shapeDropdown = Instance.new("TextButton")
shapeDropdown.Size = UDim2.new(0.5, -10, 0, 25)
shapeDropdown.Position = UDim2.new(0.5, 0, 0, 85)
shapeDropdown.BackgroundColor3 = Color3.fromRGB(50, 50, 60)
shapeDropdown.Text = "Block"
shapeDropdown.TextColor3 = Color3.fromRGB(255, 255, 255)
shapeDropdown.TextScaled = true
shapeDropdown.Font = Enum.Font.Gotham
shapeDropdown.Parent = mainFrame

local shapeCorner = Instance.new("UICorner")
shapeCorner.CornerRadius = UDim.new(0, 4)
shapeCorner.Parent = shapeDropdown

-- Material dropdown
local matLabel = Instance.new("TextLabel")
matLabel.Size = UDim2.new(0.5, -10, 0, 25)
matLabel.Position = UDim2.new(0, 10, 0, 120)
matLabel.BackgroundTransparency = 1
matLabel.Text = "Material:"
matLabel.TextColor3 = Color3.fromRGB(220, 220, 220)
matLabel.TextXAlignment = Enum.TextXAlignment.Left
matLabel.TextScaled = true
matLabel.Font = Enum.Font.Gotham
matLabel.Parent = mainFrame

local matDropdown = Instance.new("TextButton")
matDropdown.Size = UDim2.new(0.5, -10, 0, 25)
matDropdown.Position = UDim2.new(0.5, 0, 0, 120)
matDropdown.BackgroundColor3 = Color3.fromRGB(50, 50, 60)
matDropdown.Text = "Plastic"
matDropdown.TextColor3 = Color3.fromRGB(255, 255, 255)
matDropdown.TextScaled = true
matDropdown.Font = Enum.Font.Gotham
matDropdown.Parent = mainFrame

local matCorner = Instance.new("UICorner")
matCorner.CornerRadius = UDim.new(0, 4)
matCorner.Parent = matDropdown

-- Color picker button
local colorBtn = Instance.new("TextButton")
colorBtn.Size = UDim2.new(1, -20, 0, 35)
colorBtn.Position = UDim2.new(0, 10, 0, 160)
colorBtn.BackgroundColor3 = currentColor
colorBtn.Text = "Pick Color"
colorBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
colorBtn.TextScaled = true
colorBtn.Font = Enum.Font.Gotham
colorBtn.Parent = mainFrame

local colorCorner = Instance.new("UICorner")
colorCorner.CornerRadius = UDim.new(0, 4)
colorCorner.Parent = colorBtn

-- Delete last button
local deleteBtn = Instance.new("TextButton")
deleteBtn.Size = UDim2.new(0.5, -15, 0, 35)
deleteBtn.Position = UDim2.new(0, 10, 0, 210)
deleteBtn.BackgroundColor3 = Color3.fromRGB(180, 60, 60)
deleteBtn.Text = "Delete Last"
deleteBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
deleteBtn.TextScaled = true
deleteBtn.Font = Enum.Font.Gotham
deleteBtn.Parent = mainFrame

local deleteCorner = Instance.new("UICorner")
deleteCorner.CornerRadius = UDim.new(0, 4)
deleteCorner.Parent = deleteBtn

-- Delete all button
local deleteAllBtn = Instance.new("TextButton")
deleteAllBtn.Size = UDim2.new(0.5, -15, 0, 35)
deleteAllBtn.Position = UDim2.new(0.5, 5, 0, 210)
deleteAllBtn.BackgroundColor3 = Color3.fromRGB(180, 40, 40)
deleteAllBtn.Text = "Delete All"
deleteAllBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
deleteAllBtn.TextScaled = true
deleteAllBtn.Font = Enum.Font.Gotham
deleteAllBtn.Parent = mainFrame

local deleteAllCorner = Instance.new("UICorner")
deleteAllCorner.CornerRadius = UDim.new(0, 4)
deleteAllCorner.Parent = deleteAllBtn

-- Close button
local closeBtn = Instance.new("TextButton")
closeBtn.Size = UDim2.new(0, 30, 0, 30)
closeBtn.Position = UDim2.new(1, -35, 0, 2)
closeBtn.BackgroundColor3 = Color3.fromRGB(200, 60, 60)
closeBtn.Text = "X"
closeBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
closeBtn.TextScaled = true
closeBtn.Font = Enum.Font.GothamBold
closeBtn.Parent = mainFrame

local closeCorner = Instance.new("UICorner")
closeCorner.CornerRadius = UDim.new(0, 4)
closeCorner.Parent = closeBtn

-- Variables for built parts
local builtParts = {}

-- Build mode toggle
buildBtn.MouseButton1Click:Connect(function()
    buildMode = not buildMode
    buildBtn.Text = buildMode and "Build Mode: ON" or "Build Mode: OFF"
    buildBtn.BackgroundColor3 = buildMode and Color3.fromRGB(60, 120, 60) or Color3.fromRGB(60, 60, 70)
end)

-- Shape dropdown menu
local shapeList = nil
shapeDropdown.MouseButton1Click:Connect(function()
    if shapeList then shapeList:Destroy() shapeList = nil return end
    
    shapeList = Instance.new("Frame")
    shapeList.Size = UDim2.new(0, 120, 0, #shapes * 30)
    shapeList.Position = UDim2.new(1, 5, 0, 0)
    shapeList.BackgroundColor3 = Color3.fromRGB(40, 40, 50)
    shapeList.Parent = shapeDropdown
    
    local listCorner = Instance.new("UICorner")
    listCorner.CornerRadius = UDim.new(0, 4)
    listCorner.Parent = shapeList
    
    for i, shape in ipairs(shapes) do
        local btn = Instance.new("TextButton")
        btn.Size = UDim2.new(1, 0, 0, 30)
        btn.BackgroundTransparency = 0.3
        btn.Text = shape
        btn.TextColor3 = Color3.fromRGB(255, 255, 255)
        btn.TextScaled = true
        btn.Font = Enum.Font.Gotham
        btn.Parent = shapeList
        
        btn.MouseButton1Click:Connect(function()
            currentShape = shape
            shapeDropdown.Text = shape
            shapeList:Destroy()
            shapeList = nil
        end)
    end
end)

-- Material dropdown menu
local matList = nil
matDropdown.MouseButton1Click:Connect(function()
    if matList then matList:Destroy() matList = nil return end
    
    matList = Instance.new("Frame")
    matList.Size = UDim2.new(0, 120, 0, #materials * 30)
    matList.Position = UDim2.new(1, 5, 0, 0)
    matList.BackgroundColor3 = Color3.fromRGB(40, 40, 50)
    matList.Parent = matDropdown
    
    local listCorner = Instance.new("UICorner")
    listCorner.CornerRadius = UDim.new(0, 4)
    listCorner.Parent = matList
    
    for i, mat in ipairs(materials) do
        local btn = Instance.new("TextButton")
        btn.Size = UDim2.new(1, 0, 0, 30)
        btn.BackgroundTransparency = 0.3
        btn.Text = mat
        btn.TextColor3 = Color3.fromRGB(255, 255, 255)
        btn.TextScaled = true
        btn.Font = Enum.Font.Gotham
        btn.Parent = matList
        
        btn.MouseButton1Click:Connect(function()
            currentMaterial = Enum.Material[mat]
            matDropdown.Text = mat
            matList:Destroy()
            matList = nil
        end)
    end
end)

-- Color picker
colorBtn.MouseButton1Click:Connect(function()
    local colorPicker = Instance.new("ScreenGui")
    colorPicker.Name = "ColorPicker"
    colorPicker.Parent = screenGui
    
    local bg = Instance.new("Frame")
    bg.Size = UDim2.new(0, 200, 0, 250)
    bg.Position = UDim2.new(0.5, -100, 0.5, -125)
    bg.BackgroundColor3 = Color3.fromRGB(40, 40, 50)
    bg.Parent = colorPicker
    
    local bgCorner = Instance.new("UICorner")
    bgCorner.CornerRadius = UDim.new(0, 8)
    bgCorner.Parent = bg
    
    local rSlider = Instance.new("TextButton")
    rSlider.Size = UDim2.new(0.9, 0, 0, 30)
    rSlider.Position = UDim2.new(0.05, 0, 0.1, 0)
    rSlider.BackgroundColor3 = Color3.fromRGB(255, 0, 0)
    rSlider.Text = "Red"
    rSlider.TextColor3 = Color3.fromRGB(255, 255, 255)
    rSlider.Parent = bg
    
    local gSlider = Instance.new("TextButton")
    gSlider.Size = UDim2.new(0.9, 0, 0, 30)
    gSlider.Position = UDim2.new(0.05, 0, 0.25, 0)
    gSlider.BackgroundColor3 = Color3.fromRGB(0, 255, 0)
    gSlider.Text = "Green"
    gSlider.TextColor3 = Color3.fromRGB(0, 0, 0)
    gSlider.Parent = bg
    
    local bSlider = Instance.new("TextButton")
    bSlider.Size = UDim2.new(0.9, 0, 0, 30)
    bSlider.Position = UDim2.new(0.05, 0, 0.4, 0)
    bSlider.BackgroundColor3 = Color3.fromRGB(0, 0, 255)
    bSlider.Text = "Blue"
    bSlider.TextColor3 = Color3.fromRGB(255, 255, 255)
    bSlider.Parent = bg
    
    local rVal = 1
    local gVal = 1
    local bVal = 1
    
    rSlider.MouseButton1Click:Connect(function()
        rVal = (rVal % 10) + 1
        currentColor = Color3.fromRGB(rVal*25, gVal*25, bVal*25)
        colorBtn.BackgroundColor3 = currentColor
    end)
    gSlider.MouseButton1Click:Connect(function()
        gVal = (gVal % 10) + 1
        currentColor = Color3.fromRGB(rVal*25, gVal*25, bVal*25)
        colorBtn.BackgroundColor3 = currentColor
    end)
    bSlider.MouseButton1Click:Connect(function()
        bVal = (bVal % 10) + 1
        currentColor = Color3.fromRGB(rVal*25, gVal*25, bVal*25)
        colorBtn.BackgroundColor3 = currentColor
    end)
    
    local okBtn = Instance.new("TextButton")
    okBtn.Size = UDim2.new(0.8, 0, 0, 35)
    okBtn.Position = UDim2.new(0.1, 0, 0.7, 0)
    okBtn.BackgroundColor3 = Color3.fromRGB(60, 120, 60)
    okBtn.Text = "OK"
    okBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
    okBtn.Parent = bg
    
    okBtn.MouseButton1Click:Connect(function()
        colorPicker:Destroy()
    end)
end)

-- Delete last
deleteBtn.MouseButton1Click:Connect(function()
    if #builtParts > 0 then
        local last = builtParts[#builtParts]
        if last and last.Parent then last:Destroy() end
        builtParts[#builtParts] = nil
    end
end)

-- Delete all
deleteAllBtn.MouseButton1Click:Connect(function()
    for _, part in ipairs(builtParts) do
        if part and part.Parent then part:Destroy() end
    end
    builtParts = {}
end)

-- Close GUI
closeBtn.MouseButton1Click:Connect(function()
    screenGui:Destroy()
end)

-- Mouse click building
Mouse.Button1Down:Connect(function()
    if buildMode then
        local target = Mouse.Hit.Position
        local newPart = CreatePart(target)
        table.insert(builtParts, newPart)
    end
end)

-- Drag GUI
local dragging = false
local dragStart
local startPos

mainFrame.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        local mousePos = UserInputService:GetMouseLocation()
        dragStart = Vector2.new(mousePos.X, mousePos.Y)
        startPos = mainFrame.Position
        dragging = true
    end
end)

UserInputService.InputEnded:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        dragging = false
    end
end)

UserInputService.InputChanged:Connect(function(input)
    if dragging and input.UserInputType == Enum.UserInputType.MouseMovement then
        local delta = Vector2.new(input.Position.X, input.Position.Y) - dragStart
        mainFrame.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
    end
end)

print("Build cheat loaded. Press left click in build mode to build.")

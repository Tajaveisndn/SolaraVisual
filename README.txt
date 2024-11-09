--[[ 
    SolaraHub UI Library
    Version: 1.0.0
    Author: Seu Nome
]]

local Library = {
    Flags = {},
    Theme = {
        Main = Color3.fromRGB(25, 25, 25),
        Secondary = Color3.fromRGB(35, 35, 35),
        Stroke = Color3.fromRGB(50, 50, 50),
        Accent = Color3.fromRGB(44, 120, 224),
        Text = Color3.fromRGB(255, 255, 255),
        TextDark = Color3.fromRGB(150, 150, 150)
    }
}

-- Services
local UserInputService = game:GetService("UserInputService")
local TweenService = game:GetService("TweenService")
local RunService = game:GetService("RunService")
local CoreGui = game:GetService("CoreGui")

-- Variables
local Connections = {}

-- Utility Functions
local function Create(instance, properties, children)
    local object = Instance.new(instance)
    
    for i, v in pairs(properties or {}) do
        object[i] = v
    end
    
    for i, v in pairs(children or {}) do
        v.Parent = object
    end
    
    return object
end

local function MakeDraggable(topbar, frame)
    local dragging = false
    local dragInput
    local dragStart
    local startPos

    local function UpdateDrag(input)
        local delta = input.Position - dragStart
        frame.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
    end

    topbar.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 then
            dragging = true
            dragStart = input.Position
            startPos = frame.Position

            input.Changed:Connect(function()
                if input.UserInputState == Enum.UserInputState.End then
                    dragging = false
                end
            end)
        end
    end)

    topbar.InputChanged:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseMovement then
            dragInput = input
        end
    end)

    UserInputService.InputChanged:Connect(function(input)
        if input == dragInput and dragging then
            UpdateDrag(input)
        end
    end)
end

function Library:Window(title)
    local Window = {}

    -- Create Main GUI
    local SolaraHub = Create("ScreenGui", {
        Name = "SolaraHub",
        Parent = CoreGui,
        ZIndexBehavior = Enum.ZIndexBehavior.Sibling
    })

    -- Main Frame
    local Main = Create("Frame", {
        Name = "Main",
        Parent = SolaraHub,
        BackgroundColor3 = Library.Theme.Main,
        BorderSizePixel = 0,
        Position = UDim2.new(0.5, -300, 0.5, -175),
        Size = UDim2.new(0, 600, 0, 350),
        ClipsDescendants = true
    })

    -- Add Corners
    Create("UICorner", {
        CornerRadius = UDim.new(0, 6),
        Parent = Main
    })

    -- Add Stroke
    Create("UIStroke", {
        Parent = Main,
        Color = Library.Theme.Stroke,
        Thickness = 1
    })

    -- Topbar
    local Topbar = Create("Frame", {
        Name = "Topbar",
        Parent = Main,
        BackgroundColor3 = Library.Theme.Secondary,
        BorderSizePixel = 0,
        Size = UDim2.new(1, 0, 0, 40)
    })

    Create("UICorner", {
        CornerRadius = UDim.new(0, 6),
        Parent = Topbar
    })

    -- Title
    local Title = Create("TextLabel", {
        Parent = Topbar,
        BackgroundTransparency = 1,
        Position = UDim2.new(0, 15, 0, 0),
        Size = UDim2.new(0.5, -15, 1, 0),
        Font = Enum.Font.GothamBold,
        Text = title,
        TextColor3 = Library.Theme.Text,
        TextSize = 14,
        TextXAlignment = Enum.TextXAlignment.Left
    })

    -- Close Button
    local CloseButton = Create("TextButton", {
        Parent = Topbar,
        BackgroundTransparency = 1,
        Position = UDim2.new(1, -40, 0, 0),
        Size = UDim2.new(0, 40, 1, 0),
        Font = Enum.Font.GothamBold,
        Text = "×",
        TextColor3 = Library.Theme.Text,
        TextSize = 24
    })

    -- Close Button Gradient
    local CloseGradient = Create("UIGradient", {
        Color = ColorSequence.new{
            ColorSequenceKeypoint.new(0.00, Color3.fromRGB(255, 255, 255)),
            ColorSequenceKeypoint.new(1.00, Color3.fromRGB(255, 255, 255))
        },
        Parent = CloseButton
    })

    -- Close Button Animation
    CloseButton.MouseEnter:Connect(function()
        TweenService:Create(CloseGradient, TweenInfo.new(0.3), {
            Color = ColorSequence.new{
                ColorSequenceKeypoint.new(0.00, Color3.fromRGB(255, 0, 0)),
                ColorSequenceKeypoint.new(1.00, Color3.fromRGB(170, 0, 0))
            }
        }):Play()
    end)

    CloseButton.MouseLeave:Connect(function()
        TweenService:Create(CloseGradient, TweenInfo.new(0.3), {
            Color = ColorSequence.new{
                ColorSequenceKeypoint.new(0.00, Color3.fromRGB(255, 255, 255)),
                ColorSequenceKeypoint.new(1.00, Color3.fromRGB(255, 255, 255))
            }
        }):Play()
    end)

    CloseButton.MouseButton1Click:Connect(function()
        SolaraHub:Destroy()
    end)

    -- Make Window Draggable
    MakeDraggable(Topbar, Main)

    -- Tab Container
    local TabHolder = Create("Frame", {
        Name = "TabHolder",
        Parent = Main,
        BackgroundColor3 = Library.Theme.Secondary,
        BorderSizePixel = 0,
        Position = UDim2.new(0, 10, 0, 50),
        Size = UDim2.new(0, 150, 1, -60)
    })

    Create("UICorner", {
        CornerRadius = UDim.new(0, 6),
        Parent = TabHolder
    })

    local TabList = Create("ScrollingFrame", {
        Name = "TabList",
        Parent = TabHolder,
        Active = true,
        BackgroundTransparency = 1,
        Position = UDim2.new(0, 0, 0, 5),
        Size = UDim2.new(1, 0, 1, -10),
        CanvasSize = UDim2.new(0, 0, 0, 0),
        ScrollBarThickness = 2,
        ScrollBarImageColor3 = Library.Theme.Accent
    })

    Create("UIListLayout", {
        Parent = TabList,
        HorizontalAlignment = Enum.HorizontalAlignment.Center,
        SortOrder = Enum.SortOrder.LayoutOrder,
        Padding = UDim.new(0, 5)
    })

    -- Content Container
    local Container = Create("Frame", {
        Name = "Container",
        Parent = Main,
        BackgroundColor3 = Library.Theme.Secondary,
        BorderSizePixel = 0,
        Position = UDim2.new(0, 170, 0, 50),
        Size = UDim2.new(1, -180, 1, -60)
    })

    Create("UICorner", {
        CornerRadius = UDim.new(0, 6),
        Parent = Container
    })

    -- Window Methods
    function Window:Tab(name)
        local Tab = {}
        -- Tab creation code aqui...
        return Tab
    end

    return Window
end

return Library

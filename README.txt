local Library = {}

local TweenService = game:GetService("TweenService")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local Players = game:GetService("Players")
local CoreGui = game:GetService("CoreGui")

local Toggled = true
local MainFrame
local Minimized = false

function Library:CreateWindow(config)
    local title = config.Title or "SolaraHub"
    local keybind = config.Keybind or Enum.KeyCode.RightControl

    -- Criar ScreenGui
    local SolaraHub = Instance.new("ScreenGui")
    SolaraHub.Name = "SolaraHub"
    
    if syn then
        syn.protect_gui(SolaraHub)
        SolaraHub.Parent = CoreGui
    else
        SolaraHub.Parent = CoreGui
    end

    -- Main Frame
    MainFrame = Instance.new("Frame")
    local UICorner = Instance.new("UICorner")
    local TopBar = Instance.new("Frame")
    local TopBarCorner = Instance.new("UICorner")
    local Title = Instance.new("TextLabel")
    local CloseBtn = Instance.new("TextButton")
    local MinimizeBtn = Instance.new("TextButton")
    local TabHolder = Instance.new("Frame")
    local TabContainer = Instance.new("ScrollingFrame")
    local TabList = Instance.new("UIListLayout")
    local ContainerHolder = Instance.new("Frame")

    MainFrame.Name = "MainFrame"
    MainFrame.Parent = SolaraHub
    MainFrame.BackgroundColor3 = Color3.fromRGB(25, 25, 25)
    MainFrame.BorderSizePixel = 0
    MainFrame.Position = UDim2.new(0.5, -300, 0.5, -175)
    MainFrame.Size = UDim2.new(0, 600, 0, 350)
    MainFrame.ClipsDescendants = true

    UICorner.CornerRadius = UDim.new(0, 5)
    UICorner.Parent = MainFrame

    TopBar.Name = "TopBar"
    TopBar.Parent = MainFrame
    TopBar.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
    TopBar.Size = UDim2.new(1, 0, 0, 30)

    TopBarCorner.CornerRadius = UDim.new(0, 5)
    TopBarCorner.Parent = TopBar

    Title.Name = "Title"
    Title.Parent = TopBar
    Title.BackgroundTransparency = 1
    Title.Position = UDim2.new(0, 10, 0, 0)
    Title.Size = UDim2.new(0.5, 0, 1, 0)
    Title.Font = Enum.Font.GothamBold
    Title.Text = title
    Title.TextColor3 = Color3.fromRGB(255, 255, 255)
    Title.TextSize = 14
    Title.TextXAlignment = Enum.TextXAlignment.Left

    CloseBtn.Name = "CloseBtn"
    CloseBtn.Parent = TopBar
    CloseBtn.BackgroundTransparency = 1
    CloseBtn.Position = UDim2.new(1, -25, 0, 2)
    CloseBtn.Size = UDim2.new(0, 25, 0, 25)
    CloseBtn.Font = Enum.Font.GothamBold
    CloseBtn.Text = "×"
    CloseBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
    CloseBtn.TextSize = 20

    MinimizeBtn.Name = "MinimizeBtn"
    MinimizeBtn.Parent = TopBar
    MinimizeBtn.BackgroundTransparency = 1
    MinimizeBtn.Position = UDim2.new(1, -50, 0, 2)
    MinimizeBtn.Size = UDim2.new(0, 25, 0, 25)
    MinimizeBtn.Font = Enum.Font.GothamBold
    MinimizeBtn.Text = "-"
    MinimizeBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
    MinimizeBtn.TextSize = 20

    TabHolder.Name = "TabHolder"
    TabHolder.Parent = MainFrame
    TabHolder.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
    TabHolder.Position = UDim2.new(0, 5, 0, 35)
    TabHolder.Size = UDim2.new(0, 140, 1, -40)

    Instance.new("UICorner").Parent = TabHolder

    TabContainer.Name = "TabContainer"
    TabContainer.Parent = TabHolder
    TabContainer.Active = true
    TabContainer.BackgroundTransparency = 1
    TabContainer.Position = UDim2.new(0, 0, 0, 5)
    TabContainer.Size = UDim2.new(1, 0, 1, -10)
    TabContainer.ScrollBarThickness = 2
    TabContainer.CanvasSize = UDim2.new(0, 0, 0, 0)

    TabList.Parent = TabContainer
    TabList.SortOrder = Enum.SortOrder.LayoutOrder
    TabList.Padding = UDim.new(0, 5)

    ContainerHolder.Name = "ContainerHolder"
    ContainerHolder.Parent = MainFrame
    ContainerHolder.BackgroundTransparency = 1
    ContainerHolder.Position = UDim2.new(0, 150, 0, 35)
    ContainerHolder.Size = UDim2.new(1, -155, 1, -40)

    -- Make window draggable
    local dragging
    local dragInput
    local dragStart
    local startPos

    local function update(input)
        local delta = input.Position - dragStart
        MainFrame.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
    end

    TopBar.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 then
            dragging = true
            dragStart = input.Position
            startPos = MainFrame.Position

            input.Changed:Connect(function()
                if input.UserInputState == Enum.UserInputState.End then
                    dragging = false
                end
            end)
        end
    end)

    TopBar.InputChanged:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseMovement then
            dragInput = input
        end
    end)

    UserInputService.InputChanged:Connect(function(input)
        if input == dragInput and dragging then
            update(input)
        end
    end)

    -- Close and Minimize Functionality
    CloseBtn.MouseButton1Click:Connect(function()
        SolaraHub:Destroy()
    end)

    MinimizeBtn.MouseButton1Click:Connect(function()
        Minimized = not Minimized
        if Minimized then
            MainFrame:TweenSize(UDim2.new(0, 600, 0, 30), "Out", "Quad", 0.3, true)
        else
            MainFrame:TweenSize(UDim2.new(0, 600, 0, 350), "Out", "Quad", 0.3, true)
        end
    end)

    -- Toggle GUI
    UserInputService.InputBegan:Connect(function(input)
        if input.KeyCode == keybind then
            Toggled = not Toggled
            MainFrame.Visible = Toggled
        end
    end)

    local Window = {}
    
    function Window:Tab(name)
        local TabButton = Instance.new("TextButton")
        local TabButtonCorner = Instance.new("UICorner")
        local Container = Instance.new("ScrollingFrame")
        local ContainerList = Instance.new("UIListLayout")

        TabButton.Name = name
        TabButton.Parent = TabContainer
        TabButton.BackgroundColor3 = Color3.fromRGB(35, 35, 35)
        TabButton.Size = UDim2.new(1, -10, 0, 30)
        TabButton.Font = Enum.Font.Gotham
        TabButton.Text = name
        TabButton.TextColor3 = Color3.fromRGB(255, 255, 255)
        TabButton.TextSize = 14
        TabButton.AutoButtonColor = false

        TabButtonCorner.CornerRadius = UDim.new(0, 5)
        TabButtonCorner.Parent = TabButton

        Container.Name = name.."Container"
        Container.Parent = ContainerHolder
        Container.Active = true
        Container.BackgroundTransparency = 1
        Container.Size = UDim2.new(1, 0, 1, 0)
        Container.ScrollBarThickness = 2
        Container.Visible = false

        ContainerList.Parent = Container
        ContainerList.SortOrder = Enum.SortOrder.LayoutOrder
        ContainerList.Padding = UDim.new(0, 5)

        if #TabContainer:GetChildren() == 2 then
            Container.Visible = true
            TabButton.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
        end

        TabButton.MouseButton1Click:Connect(function()
            for _, container in pairs(ContainerHolder:GetChildren()) do
                container.Visible = false
            end
            for _, button in pairs(TabContainer:GetChildren()) do
                if button:IsA("TextButton") then
                    button.BackgroundColor3 = Color3.fromRGB(35, 35, 35)
                end
            end
            Container.Visible = true
            TabButton.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
        end)

        local Tab = {}

        -- Add elements functions here (Button, Toggle, etc.)
        function Tab:Button(text, callback)
            local Button = Instance.new("TextButton")
            local ButtonCorner = Instance.new("UICorner")

            Button.Name = text
            Button.Parent = Container
            Button.BackgroundColor3 = Color3.fromRGB(35, 35, 35)
            Button.Size = UDim2.new(1, -10, 0, 30)
            Button.Font = Enum.Font.Gotham
            Button.Text = text
            Button.TextColor3 = Color3.fromRGB(255, 255, 255)
            Button.TextSize = 14
            Button.AutoButtonColor = false

            ButtonCorner.CornerRadius = UDim.new(0, 5)
            ButtonCorner.Parent = Button

            Container.CanvasSize = UDim2.new(0, 0, 0, ContainerList.AbsoluteContentSize.Y + 5)

            Button.MouseButton1Click:Connect(function()
                callback()
            end)
        end

        function Tab:Toggle(text, default, callback)
            local Toggle = Instance.new("TextButton")
            local ToggleCorner = Instance.new("UICorner")
            local Status = Instance.new("Frame")
            local StatusCorner = Instance.new("UICorner")

            Toggle.Name = text
            Toggle.Parent = Container
            Toggle.BackgroundColor3 = Color3.fromRGB(35, 35, 35)
            Toggle.Size = UDim2.new(1, -10, 0, 30)
            Toggle.Font = Enum.Font.Gotham
            Toggle.Text = "  "..text
            Toggle.TextColor3 = Color3.fromRGB(255, 255, 255)
            Toggle.TextSize = 14
            Toggle.TextXAlignment = Enum.TextXAlignment.Left
            Toggle.AutoButtonColor = false

            ToggleCorner.CornerRadius = UDim.new(0, 5)
            ToggleCorner.Parent = Toggle

            Status.Name = "Status"
            Status.Parent = Toggle
            Status.AnchorPoint = Vector2.new(1, 0.5)
            Status.BackgroundColor3 = default and Color3.fromRGB(0, 255, 0) or Color3.fromRGB(255, 0, 0)
            Status.Position = UDim2.new(1, -5, 0.5, 0)
            Status.Size = UDim2.new(0, 20, 0, 20)

            StatusCorner.CornerRadius = UDim.new(0, 5)
            StatusCorner.Parent = Status

            local toggled = default

            Container.CanvasSize = UDim2.new(0, 0, 0, ContainerList.AbsoluteContentSize.Y + 5)

            Toggle.MouseButton1Click:Connect(function()
                toggled = not toggled
                Status.BackgroundColor3 = toggled and Color3.fromRGB(0, 255, 0) or Color3.fromRGB(255, 0, 0)
                callback(toggled)
            end)
        end

        return Tab
    end
    
    return Window
end

return Library

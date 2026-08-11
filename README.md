--// Rainbow 2D ESP
--// Tool Images + Small Name + Small HP + 2D Box + Line + Movable Menu

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")

local LocalPlayer = Players.LocalPlayer
local PlayerGui = LocalPlayer:WaitForChild("PlayerGui")

local ESPEnabled = true
local Objects = {}

--==================================================
-- Rainbow
--==================================================

local function Rainbow()
	return Color3.fromHSV((tick() % 5) / 5, 1, 1)
end

--==================================================
-- إنشاء الخط
--==================================================

local function SetLine(line, from, to, thickness)

	local difference = to - from
	local length = difference.Magnitude

	line.Position = UDim2.fromOffset(
		(from.X + to.X) / 2,
		(from.Y + to.Y) / 2
	)

	line.Size = UDim2.fromOffset(
		length,
		thickness
	)

	line.Rotation = math.deg(
		math.atan2(difference.Y, difference.X)
	)
end

--==================================================
-- الحصول على الأدوات
--==================================================

local function GetTools(player)

	local tools = {}

	local backpack = player:FindFirstChildOfClass("Backpack")

	if backpack then

		for _, object in ipairs(backpack:GetChildren()) do

			if object:IsA("Tool") then
				table.insert(tools, object)
			end

		end
	end

	local character = player.Character

	if character then

		for _, object in ipairs(character:GetChildren()) do

			if object:IsA("Tool") then
				table.insert(tools, object)
			end

		end
	end

	return tools
end

--==================================================
-- تحديث صور الأدوات
--==================================================

local function UpdateToolImages(data, player, rainbow)

	for _, image in ipairs(data.ToolImages) do
		image:Destroy()
	end

	table.clear(data.ToolImages)

	local textures = {}

	for _, tool in ipairs(GetTools(player)) do

		local texture = tool.TextureId

		if texture and texture ~= "" then
			table.insert(textures, texture)
		end

	end

	-- الحد الأقصى 6 صور
	local count = math.min(#textures, 6)

	if count == 0 then
		data.ToolsFrame.Visible = false
		return
	end

	data.ToolsFrame.Visible = true

	local imageSize = 24
	local gap = 4
	local totalWidth =
		count * imageSize +
		(count - 1) * gap

	data.ToolsFrame.Size =
		UDim2.fromOffset(
			totalWidth,
			imageSize
		)

	for i = 1, count do

		local image = Instance.new("ImageLabel")

		image.BackgroundTransparency = 1

		image.Size =
			UDim2.fromOffset(
				imageSize,
				imageSize
			)

		image.Position =
			UDim2.fromOffset(
				(i - 1) *
				(imageSize + gap),
				0
			)

		image.Image =
			textures[i]

		image.ImageColor3 =
			rainbow

		image.Parent =
			data.ToolsFrame

		table.insert(
			data.ToolImages,
			image
		)
	end
end

--==================================================
-- إنشاء ESP
--==================================================

local function CreateESP(player)

	if player == LocalPlayer then
		return
	end

	local container =
		Instance.new("Frame")

	container.Name =
		"ESP_" .. player.Name

	container.BackgroundTransparency = 1

	container.Size =
		UDim2.fromScale(1,1)

	container.Visible = false

	container.Parent =
		PlayerGui:WaitForChild(
			"RainbowESP"
		)

	--==================================================
	-- 2D BOX
	--==================================================

	local box =
		Instance.new("Frame")

	box.Name =
		"2DBox"

	box.BackgroundTransparency = 1
	box.BorderSizePixel = 0

	box.Visible = false

	box.Parent =
		container

	local top =
		Instance.new("Frame")

	local bottom =
		Instance.new("Frame")

	local left =
		Instance.new("Frame")

	local right =
		Instance.new("Frame")

	for _, side in ipairs({
		top,
		bottom,
		left,
		right
	}) do

		side.BackgroundColor3 =
			Color3.new(1,1,1)

		side.BorderSizePixel = 0

		side.Parent =
			box
	end

	top.Size =
		UDim2.new(1,0,0,2)

	bottom.Size =
		UDim2.new(1,0,0,2)

	bottom.Position =
		UDim2.new(0,0,1,-2)

	left.Size =
		UDim2.new(0,2,1,0)

	right.Size =
		UDim2.new(0,2,1,0)

	right.Position =
		UDim2.new(1,-2,0,0)

	--==================================================
	-- صور الأدوات
	--==================================================

	local toolsFrame =
		Instance.new("Frame")

	toolsFrame.Name =
		"ToolImages"

	toolsFrame.BackgroundTransparency = 1

	toolsFrame.AnchorPoint =
		Vector2.new(0.5,0.5)

	toolsFrame.Visible = false

	toolsFrame.Parent =
		container

	--==================================================
	-- الاسم
	--==================================================

	local nameLabel =
		Instance.new("TextLabel")

	nameLabel.BackgroundTransparency = 1

	nameLabel.TextSize = 10

	nameLabel.TextColor3 =
		Color3.new(1,1,1)

	nameLabel.TextStrokeTransparency = 0

	nameLabel.Font =
		Enum.Font.GothamBold

	nameLabel.AnchorPoint =
		Vector2.new(0.5,0)

	nameLabel.Parent =
		container

	--==================================================
	-- HP
	--==================================================

	local healthLabel =
		Instance.new("TextLabel")

	healthLabel.BackgroundTransparency = 1

	healthLabel.TextSize = 9

	healthLabel.TextColor3 =
		Color3.new(1,1,1)

	healthLabel.TextStrokeTransparency = 0

	healthLabel.Font =
		Enum.Font.GothamBold

	healthLabel.AnchorPoint =
		Vector2.new(0.5,0)

	healthLabel.Parent =
		container

	--==================================================
	-- LINE
	--==================================================

	local line =
		Instance.new("Frame")

	line.BackgroundColor3 =
		Color3.new(1,1,1)

	line.BorderSizePixel = 0

	line.AnchorPoint =
		Vector2.new(0.5,0.5)

	line.Visible = false

	line.Parent =
		container

	Objects[player] = {

		Container = container,

		Box = box,

		Top = top,
		Bottom = bottom,
		Left = left,
		Right = right,

		ToolsFrame = toolsFrame,

		ToolImages = {},

		Name = nameLabel,

		Health = healthLabel,

		Line = line,

		LastToolUpdate = 0
	}
end

--==================================================
-- GUI
--==================================================

local espGui =
	Instance.new("ScreenGui")

espGui.Name =
	"RainbowESP"

espGui.ResetOnSpawn = false

espGui.IgnoreGuiInset = true

espGui.Parent =
	PlayerGui

--==================================================
-- اللاعبين
--==================================================

for _, player in ipairs(
	Players:GetPlayers()
) do

	CreateESP(player)

end

Players.PlayerAdded:Connect(
	function(player)
		CreateESP(player)
	end
)

Players.PlayerRemoving:Connect(
	function(player)

		local data =
			Objects[player]

		if data then

			data.Container:Destroy()

			Objects[player] = nil

		end
	end
)

--==================================================
-- تحديث ESP
--==================================================

RunService.RenderStepped:Connect(
	function()

		local Camera =
			workspace.CurrentCamera

		if not Camera then
			return
		end

		local rainbow =
			Rainbow()

		local myCharacter =
			LocalPlayer.Character

		local myRoot =
			myCharacter
			and myCharacter:FindFirstChild(
				"HumanoidRootPart"
			)

		for player, data in pairs(
			Objects
		) do

			local character =
				player.Character

			local root =
				character
				and character:FindFirstChild(
					"HumanoidRootPart"
				)

			local head =
				character
				and character:FindFirstChild(
					"Head"
				)

			local humanoid =
				character
				and character:FindFirstChildOfClass(
					"Humanoid"
				)

			if not ESPEnabled
				or not character
				or not root
				or not head
				or not humanoid
				or humanoid.Health <= 0 then

				data.Container.Visible = false
				data.Line.Visible = false

				continue
			end

			--==================================================
			-- مواقع اللاعب على الشاشة
			--==================================================

			local headPosition, headVisible =
				Camera:WorldToViewportPoint(
					head.Position +
					Vector3.new(0,0.4,0)
				)

			local footPosition, footVisible =
				Camera:WorldToViewportPoint(
					root.Position -
					Vector3.new(0,3,0)
				)

			if not headVisible
				and not footVisible then

				data.Container.Visible = false

				continue
			end

			data.Container.Visible = true

			--==================================================
			-- حجم المربع
			--==================================================

			local height =
				math.abs(
					headPosition.Y -
					footPosition.Y
				)

			local width =
				height * 0.55

			local x =
				headPosition.X

			local y =
				(
					headPosition.Y +
					footPosition.Y
				) / 2

			data.Box.Position =
				UDim2.fromOffset(
					x - width/2,
					y - height/2
				)

			data.Box.Size =
				UDim2.fromOffset(
					width,
					height
				)

			data.Box.Visible = true

			-- Rainbow للمربع

			data.Top.BackgroundColor3 =
				rainbow

			data.Bottom.BackgroundColor3 =
				rainbow

			data.Left.BackgroundColor3 =
				rainbow

			data.Right.BackgroundColor3 =
				rainbow

			--==================================================
			-- صور الأدوات فوق الرأس
			--==================================================

			if tick() -
				data.LastToolUpdate >
				0.25 then

				UpdateToolImages(
					data,
					player,
					rainbow
				)

				data.LastToolUpdate =
					tick()

			else

				for _, image in ipairs(
					data.ToolImages
				) do

					image.ImageColor3 =
						rainbow
				end
			end

			local toolY =
				y -
				height/2 -
				45

			data.ToolsFrame.Position =
				UDim2.fromOffset(
					x,
					toolY
				)

			--==================================================
			-- الاسم
			--==================================================

			data.Name.Text =
				player.DisplayName

			data.Name.TextColor3 =
				rainbow

			data.Name.Position =
				UDim2.fromOffset(
					x,
					toolY + 14
				)

			data.Name.Size =
				UDim2.fromOffset(
					120,
					14
				)

			--==================================================
			-- HP
			--==================================================

			local hp =
				math.floor(
					humanoid.Health
				)

			local maxHp =
				math.floor(
					humanoid.MaxHealth
				)

			data.Health.Text =
				"HP: " ..
				hp ..
				"/" ..
				maxHp

			data.Health.TextColor3 =
				rainbow

			data.Health.Position =
				UDim2.fromOffset(
					x,
					toolY + 28
				)

			data.Health.Size =
				UDim2.fromOffset(
					120,
					14
				)

			--==================================================
			-- الخط
			--==================================================

			if myRoot then

				local startPosition =
					Camera:WorldToViewportPoint(
						myRoot.Position
					)

				local startVector =
					Vector2.new(
						startPosition.X,
						startPosition.Y
					)

				local targetVector =
					Vector2.new(
						x,
						y
					)

				SetLine(
					data.Line,
					startVector,
					targetVector,
					2
				)

				data.Line.BackgroundColor3 =
					rainbow

				data.Line.Visible = true

			else

				data.Line.Visible = false

			end
		end
	end
)

--==================================================
-- القائمة المتحركة
--==================================================

local menu =
	Instance.new("Frame")

menu.Name = "Menu"

menu.Size =
	UDim2.fromOffset(
		170,
		85
	)

menu.Position =
	UDim2.new(
		0,
		20,
		0.4,
		0
	)

menu.BackgroundColor3 =
	Color3.fromRGB(
		25,
		25,
		25
	)

menu.BorderSizePixel = 0

menu.Active = true

menu.Draggable = true

menu.Parent =
	espGui

local corner =
	Instance.new("UICorner")

corner.CornerRadius =
	UDim.new(0,10)

corner.Parent =
	menu

--==================================================
-- العنوان
--==================================================

local title =
	Instance.new("TextLabel")

title.Size =
	UDim2.new(
		1,
		0,
		0,
		28
	)

title.BackgroundTransparency = 1

title.Text =
	"Rainbow ESP"

title.TextColor3 =
	Color3.new(1,1,1)

title.TextSize = 17

title.Font =
	Enum.Font.GothamBold

title.Parent =
	menu

--==================================================
-- زر التشغيل
--==================================================

local button =
	Instance.new("TextButton")

button.Position =
	UDim2.new(
		0.1,
		0,
		0.48,
		0
	)

button.Size =
	UDim2.new(
		0.8,
		0,
		0,
		32
	)

button.Text =
	"ESP: ON"

button.TextColor3 =
	Color3.new(1,1,1)

button.TextSize = 15

button.Font =
	Enum.Font.GothamBold

button.BackgroundColor3 =
	Color3.fromRGB(
		0,
		170,
		0
	)

button.Parent =
	menu

local buttonCorner =
	Instance.new("UICorner")

buttonCorner.CornerRadius =
	UDim.new(0,8)

buttonCorner.Parent =
	button

button.MouseButton1Click:Connect(
	function()

		ESPEnabled =
			not ESPEnabled

		if ESPEnabled then

			button.Text =
				"ESP: ON"

			button.BackgroundColor3 =
				Color3.fromRGB(
					0,
					170,
					0
				)

		else

			button.Text =
				"ESP: OFF"

			button.BackgroundColor3 =
				Color3.fromRGB(
					170,
					0,
					0
				)

		end
	end
)

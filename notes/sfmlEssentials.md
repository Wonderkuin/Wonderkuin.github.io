# SFML Essentials

### [<主页](https://www.wangdekui.com/)

## Chapter 1

### Window and Handle Input

```c++
#include <SFML/Window.hpp>
#include <iostream>
int main()
{
	sf::VideoMode video = sf::VideoMode(300, 200, 32U);
	//sf::VideoMode video = sf::VideoMode::getDesktopMode();

	std::vector<sf::VideoMode> videoModes = video.getFullscreenModes();

	for (auto vm : videoModes)
		std::cout << vm.width << " " << vm.height << std::endl;

	if (video.isValid())
		std::cout << "VideoMode is Valid." << std::endl;

	sf::Uint32 style = sf::Style::Close | sf::Style::Titlebar | sf::Style::Resize;

	sf::Uint32 depthBits = 0U;
	sf::Uint32 stencilBits = 0U;
	sf::Uint32 antialiasingLevel = 0U;
	sf::Uint32 majorVersion = 0U;
	sf::Uint32 minorVersion = 0U;
	const sf::ContextSettings setting = sf::ContextSettings(depthBits, stencilBits, antialiasingLevel, majorVersion, minorVersion);

	sf::Window window(video, "The Title", style, setting);

	window.setMouseCursorVisible(false);

	sf::sleep(sf::seconds(3));

	// Handle input
	sf::String buffer;
	while (window.isOpen())
	{
		sf::Event event;
		//while (window.waitEvent(event)); //use in threads
		while (window.pollEvent(event))
		{
			switch (event.type)
			{
			case sf::Event::EventType::Closed:
				window.close();
				break;
			//case sf::Event::EventType::KeyPressed:
			//	if (event.key.code == sf::Keyboard::Key::Space)
			//		window.setTitle("Space perssed");
			//	break;
			//case sf::Event::EventType::KeyReleased:
			//	if (event.key.code == sf::Keyboard::Key::Space)
			//		window.setTitle("Space released");
			//	else if (event.key.code == sf::Keyboard::Key::Escape)
			//		window.close();
			//	break;
			case sf::Event::EventType::TextEntered:
				buffer += event.text.unicode;
				break;
			case sf::Event::EventType::KeyReleased:
				if (event.key.code == sf::Keyboard::Key::Return)
				{
					window.setTitle(buffer);
					buffer.clear();
				}
				break;
			default:
				break;
			}
		}
	}
}
```

### Shape rendering and transformations

```c++
#include <SFML/Graphics.hpp>

int main()
{
	sf::RenderWindow window(sf::VideoMode(640, 480), "Title");

	window.setFramerateLimit(30);

	sf::CircleShape circleShape(50);
	circleShape.setFillColor(sf::Color::Red);
	circleShape.setOutlineColor(sf::Color::White);
	circleShape.setOutlineThickness(3);

	sf::ConvexShape triangle;
	triangle.setPointCount(3);
	triangle.setPoint(0, sf::Vector2f(100, 0));
	triangle.setPoint(1, sf::Vector2f(100, 100));
	triangle.setPoint(2, sf::Vector2f(0, 100));
	triangle.setFillColor(sf::Color::Blue);

	sf::RectangleShape rectShape(sf::Vector2f(50, 50));
	rectShape.setFillColor(sf::Color::Magenta);

	sf::RectangleShape rect(sf::Vector2f(50, 50));
	rect.setFillColor(sf::Color::Red);
	//rect.setPosition(sf::Vector2f(50, 50));
	rect.move(sf::Vector2f(50, 50));
	rect.setRotation(30);
	rect.setScale(sf::Vector2f(2, 1));

	while (window.isOpen())
	{
		//Handle events
		sf::Event event;
		while (window.pollEvent(event))
		{
			if (event.type == sf::Event::EventType::Closed)
				window.close();
		}

		//Update scene

		//Render cycle
		window.clear(sf::Color::Black);

		window.draw(circleShape);
		window.draw(rectShape);
		window.draw(triangle);
		window.draw(rect);

		//Render objects

		window.display();
	}

	return 0;
}
```

### Simple Move

```c++
#include <SFML/Graphics.hpp>

int main()
{
	sf::RenderWindow window(sf::VideoMode(480, 180), "Animation");

	window.setFramerateLimit(60);

	sf::RectangleShape rect(sf::Vector2f(50, 50));
	rect.setFillColor(sf::Color::Red);
	rect.setOrigin(sf::Vector2f(25, 25));
	rect.setPosition(sf::Vector2f(50, 50));

	bool moving = false;
	while (window.isOpen())
	{
		//Handle events
		/*sf::Event ev;
		while (window.pollEvent(ev))
		{
			if (ev.type == sf::Event::EventType::KeyPressed &&
				ev.key.code == sf::Keyboard::Right)
				moving = true;
			if (ev.type == sf::Event::EventType::KeyReleased &&
				ev.key.code == sf::Keyboard::Right)
				moving = false;
		}*/

		//sf::Mouse::getPosition();
		//sf::Mouse::setPosition(sf::Vector2i(1, 1));
		//sf::Mouse::isButtonPressed(sf::Mouse::Button::Right);
		//
		//sf::Joystick::isConnected(1U);

		moving = sf::Keyboard::isKeyPressed(sf::Keyboard::Key::Right);

		//Update frame
		if (moving)
		{
			rect.rotate(1.5f);
			rect.move(sf::Vector2f(1, 0));
		}

		//Render frame
		window.clear(sf::Color::Black);
		window.draw(rect);
		window.display();
	}

	return 0;
}
```

### Simple Game

```c++
#include <SFML/Graphics.hpp>


void initShape(sf::RectangleShape& shape, sf::Vector2f const& pos, sf::Color const& color)
{
	shape.setFillColor(color);
	shape.setPosition(pos);
	shape.setOrigin(shape.getSize() * 0.5f); // center of rectangle
}

int main()
{
	sf::RenderWindow window(sf::VideoMode(480, 180), "Bad Squares");
	window.setFramerateLimit(60); // target frames per second

	sf::Vector2f startPos = sf::Vector2f(50, 50);
	sf::RectangleShape playerRect(sf::Vector2f(50, 50));
	initShape(playerRect, startPos, sf::Color::Green);
	sf::RectangleShape targetRect(sf::Vector2f(50, 50));
	initShape(targetRect, sf::Vector2f(400, 50), sf::Color::Blue);
	sf::RectangleShape badRect(sf::Vector2f(50, 100));
	initShape(badRect, sf::Vector2f(250, 50), sf::Color::Red);

	while (window.isOpen())
	{
		playerRect.move(1, 0);
		if (sf::Keyboard::isKeyPressed(sf::Keyboard::Key::Down))
			playerRect.move(0, 1);
		if (sf::Keyboard::isKeyPressed(sf::Keyboard::Key::Up))
			playerRect.move(0, -1);

		//playerRect.getLocalBounds().intersects(targetRect.getLocalBounds());

		if (playerRect.getGlobalBounds().intersects(targetRect.getGlobalBounds()))
			window.close();
		if (playerRect.getGlobalBounds().intersects(badRect.getGlobalBounds()))
			playerRect.setPosition(startPos);

		window.clear(sf::Color::Black);

		window.draw(playerRect);
		window.draw(targetRect);
		window.draw(badRect);

		window.display();
	}

	return 0;
}

```

## Chapter 2


### image加载
```c++
#include <SFML/Graphics.hpp>

int main()
{
	sf::RenderWindow window(sf::VideoMode(480, 180), "Bad Squares");
	window.setFramerateLimit(60); // target frames per second

	// Color
	sf::Image image1;
	image1.create(50, 50, sf::Color::Red);

	// Pixels
	const unsigned int kWidth = 5, kHeight = 5;
	sf::Uint8 pixels[kWidth * kHeight * 4];
	for (int index = 0, i = 0; i < kWidth; i++)
		for (int j = 0; j < kHeight; j++)
		{
			pixels[index++] = 255;
			pixels[index++] = 0;
			pixels[index++] = 0;
			pixels[index++] = 255;
		}
	sf::Image image2;
	image2.create(kWidth, kHeight, pixels);

	auto pixel_11 = image2.getPixel(1, 1);
	image2.setPixel(1, 1, sf::Color::Yellow);

	const sf::Uint8* pixelsPtr = image2.getPixelsPtr();

	image2.flipHorizontally();

	image2.saveToFile("./img2.png");

	// Image
	sf::Image image3;
	if (!image3.loadFromFile("leaf.png"))
		return -1;

	while (window.isOpen())
	{

		window.clear(sf::Color::Black);

		//window.draw();

		window.display();
	}

	return 0;
}

```

### texture加载
```c++
#include <SFML/Graphics.hpp>

int main()
{
	sf::RenderWindow window(sf::VideoMode(1280, 800), "Bad Squares");
	window.setFramerateLimit(60); // target frames per second

	unsigned int maxSize = sf::Texture::getMaximumSize();//texture.create(

	sf::Texture texture;
	//if (!texture.loadFromFile("leaf.png", sf::IntRect(0, 0, 320, 100)))
	if (!texture.loadFromFile("leaf.png"))
		return -1;

	sf::Image image;
	image.create(50, 50, sf::Color::Red);
	sf::Texture texture2;
	texture2.loadFromImage(image);

	sf::Image image2;
	image2 = texture2.copyToImage(); // From GPU to RAM

	sf::Vector2u textureSize = texture.getSize();
	float rectWidth = static_cast<float>(textureSize.x);
	float rectHeight = static_cast<float>(textureSize.y);
	sf::RectangleShape rectShape(sf::Vector2f(rectWidth, rectHeight));
	rectShape.setTexture(&texture);

	while (window.isOpen())
	{

		window.clear(sf::Color::Black);
		window.draw(rectShape);
		window.display();
	}

	return 0;
}
```

### 显示一部分texture
```c++
#include <SFML/Graphics.hpp>

int main()
{
	sf::RenderWindow window(sf::VideoMode(400, 400), "Bad Squares");
	window.setFramerateLimit(60); // target frames per second

	sf::Texture texture;
	if (!texture.loadFromFile("leaf.png"))
		return -1;

	sf::ConvexShape shape(5);
	shape.setPoint(0, sf::Vector2f(0, 0));
	shape.setPoint(1, sf::Vector2f(200, 0));
	shape.setPoint(2, sf::Vector2f(180, 120));
	shape.setPoint(3, sf::Vector2f(100, 200));
	shape.setPoint(4, sf::Vector2f(20, 120));
	shape.setTexture(&texture);
	shape.setOutlineThickness(2);
	shape.setOutlineColor(sf::Color::Red);
	shape.move(20, 20); //Move it, so the outline is clearly visible

	while (window.isOpen())
	{
		window.clear(sf::Color::Black);
		window.draw(shape);
		window.display();
	}

	return 0;
}

```

### 裁切texture

```c++
#include <SFML/Graphics.hpp>

int main()
{
	sf::RenderWindow window(sf::VideoMode(800, 800), "Bad Squares");
	window.setFramerateLimit(60); // target frames per second

	sf::Texture texture;
	texture.loadFromFile("tile.png");
	texture.setRepeated(true);
	//texture.setSmooth(true);

	sf::RectangleShape rectShape(sf::Vector2f(128 * 3, 221 * 2));
	rectShape.setTextureRect(sf::IntRect(0, 0, 128 * 3, 221 * 2));
	rectShape.setTexture(&texture);

	while (window.isOpen())
	{
		window.clear(sf::Color::Black);
		window.draw(rectShape);
		window.display();
	}

	return 0;
}
```

## [<主页](https://www.wangdekui.com/)
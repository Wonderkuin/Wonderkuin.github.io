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

### 资源管理

```c++
//AssetManager.h
#ifndef ASSET_MANAGER_H
#define ASSET_MANAGER_H

#include <SFML/Graphics.hpp>
#include <map>

class AssetManager
{
public:
	AssetManager();

	static sf::Texture& GetTexture(std::string const& filename);

private:
	std::map<std::string, sf::Texture> m_Textures;

	static AssetManager* sInstance;
};

#endif

//AssetManager.cpp
#include "AssetManager.h"
#include <assert.h>

AssetManager* AssetManager::sInstance = nullptr;

AssetManager::AssetManager()
{
	assert(sInstance == nullptr);
	sInstance = this;
}

sf::Texture& AssetManager::GetTexture(std::string const& filename)
{
	auto& texMap = sInstance->m_Textures;

	auto pairFound = texMap.find(filename);

	if (pairFound != texMap.end())
		return pairFound->second;
	else
	{
		auto& texture = texMap[filename];
		texture.loadFromFile(filename);
		return texture;
	}
}

//main.cpp
#include <SFML/Graphics.hpp>
#include "AssetManager.h"

int main()
{
	sf::RenderWindow window(sf::VideoMode(640, 480), "AssetManager");
	AssetManager manager;

	sf::Sprite sprite1 = sf::Sprite(AssetManager::GetTexture("egg.png"));
	sf::Sprite sprite2 = sf::Sprite(AssetManager::GetTexture("egg.png"));
	sf::Sprite sprite3 = sf::Sprite(AssetManager::GetTexture("egg.png"));

	sprite1.setScale(sf::Vector2f(0.5, 0.5));

	while (window.isOpen())
	{
		window.clear(sf::Color::Black);
		window.draw(sprite1);
		window.display();
	}

	return 0;
}

```

### Time

```c++
#include <SFML/Graphics.hpp>
#include <iostream>

int main()
{
	sf::Time time = sf::seconds(5) + sf::milliseconds(100);
	if (time > sf::seconds(5.09))
		std::cout << "It works";

	sf::RenderWindow window(sf::VideoMode(640, 480), "Time");

	sf::Time elapsedTime;
	sf::Time deltaTime;
	sf::Clock clock;
	while (window.isOpen())
	{
		//deltaTime = clock.getElapsedTime();
		deltaTime = clock.restart();
		elapsedTime += deltaTime;

		if (elapsedTime > sf::seconds(5))
			window.close();

		float dtAsSeconds = deltaTime.asSeconds();

		window.clear(sf::Color::Black);
		window.display();
	}
}
```

### Animation

```c++
#include <SFML/Graphics.hpp>
#include "AssetManager.h"

int main()
{
	sf::RenderWindow window(sf::VideoMode(640, 480), "Time");
	AssetManager am;

	sf::Vector2i spriteSize(32, 32);
	sf::Sprite sprite(am.GetTexture("crystal/crystal-qubodup-ccby3-32-green.png"));
	sprite.setTextureRect(sf::IntRect(0, 0, spriteSize.x, spriteSize.y));

	int framesNum = 8;
	float animationDuration = 1;

	sf::Time elapsedTime;
	sf::Clock clock;
	while (window.isOpen())
	{
		sf::Time deltaTime = clock.restart();

		elapsedTime += deltaTime;
		float timeAsSeconds = elapsedTime.asSeconds();

		int animFrame = static_cast<int>((timeAsSeconds / animationDuration) * framesNum) % framesNum;

		sprite.setTextureRect(sf::IntRect(animFrame * spriteSize.x, 0, spriteSize.x, spriteSize.y));

		window.clear(sf::Color::Black);
		window.draw(sprite);
		window.display();
	}
}
```


### Animator

```c++
// Animator.h
#ifndef ANIMATOR_H
#define ANIMATOR_H

#include <SFML/Graphics.hpp>
#include <list>
#include <string>
#include <vector>

class Animator
{
public:
	struct Animation
	{
		std::string m_Name;
		std::string m_TextureName;
		std::vector<sf::IntRect> m_Frames;
		sf::Time m_Duration;
		bool m_Looping;

		Animation(std::string const& name, std::string const& textureName,
			sf::Time const& duration, bool looping)
			: m_Name(name), m_TextureName(textureName),
			m_Duration(duration), m_Looping(looping)
		{	}

		void AddFrames(sf::Vector2i const& startFrom, sf::Vector2i const& frameSize, unsigned int frames)
		{
			sf::Vector2i current = startFrom;
			for (unsigned int i = 0; i < frames; i++)
			{
				m_Frames.push_back(sf::IntRect(current.x, current.y, frameSize.x, frameSize.y));
				current.x += frameSize.x;
			}
		}
	};

	Animator(sf::Sprite& sprite);

	Animator::Animation& CreateAnimation(std::string const& name,
		std::string const& textureName, sf::Time const& duration, bool loop = false);

	void Update(sf::Time const& dt);

	bool SwitchAnimation(std::string const& name);

	std::string GetCurrentAnimationName() const;
private:
	Animator::Animation* FindAnimation(std::string const& name);

	void SwitchAnimation(Animator::Animation* animation);

	sf::Sprite& m_Sprite;
	sf::Time m_CurrentTime;
	std::list<Animator::Animation> m_Animations;
	Animator::Animation* m_CurrentAnimation;
};

#endif

// Animator.cpp
#include "Animator.h"
#include "AssetManager.h"

Animator::Animator(sf::Sprite& sprite)
	:m_Sprite(sprite), m_CurrentTime(), m_CurrentAnimation(nullptr)
{
}

Animator::Animation& Animator::CreateAnimation(std::string const& name,
	std::string const& textureName, sf::Time const& duration, bool loop)
{
	m_Animations.push_back(
		Animator::Animation(name, textureName, duration, loop));

	if (m_CurrentAnimation == nullptr)
		SwitchAnimation(&m_Animations.back());

	return m_Animations.back();
}

void Animator::SwitchAnimation(Animator::Animation* animation)
{
	if (animation != nullptr)
	{
		m_Sprite.setTexture(AssetManager::GetTexture(animation->m_TextureName));
	}

	m_CurrentAnimation = animation;
	m_CurrentTime = sf::Time::Zero;
}

bool Animator::SwitchAnimation(std::string const& name)
{
	auto animation = FindAnimation(name);
	if (animation != nullptr)
	{
		SwitchAnimation(animation);
		return true;
	}

	return false;
}

Animator::Animation* Animator::FindAnimation(std::string const& name)
{
	for (auto it = m_Animations.begin(); it != m_Animations.end(); ++it)
	{
		if (it->m_Name == name)
			return &*it;
	}

	return nullptr;
}

std::string Animator::GetCurrentAnimationName() const
{
	if (m_CurrentAnimation != nullptr)
		return m_CurrentAnimation->m_Name;

	return "";
}

void Animator::Update(sf::Time const& dt)
{
	if (m_CurrentAnimation == nullptr)
		return;

	m_CurrentTime += dt;

	float scaledTime = (m_CurrentTime.asSeconds() / m_CurrentAnimation->m_Duration.asSeconds());
	int numFrames = m_CurrentAnimation->m_Frames.size();
	int currentFrame = static_cast<int>(scaledTime * numFrames);

	if (m_CurrentAnimation->m_Looping)
		currentFrame %= numFrames;
	else if (currentFrame >= numFrames)
		currentFrame = numFrames;

	m_Sprite.setTextureRect(m_CurrentAnimation->m_Frames[currentFrame]);
}

// main.cpp
#include <SFML/Graphics.hpp>
#include "AssetManager.h"
#include "Animator.h"

int main()
{
	sf::RenderWindow window(sf::VideoMode(640, 480), "Time");
	AssetManager am;

	sf::Vector2i spriteSize(32, 32);
	sf::Sprite sprite;
	Animator animator(sprite);
	auto& idleAnimation = animator.CreateAnimation("idle", "crystal.png", sf::seconds(1), true);
	idleAnimation.AddFrames(sf::Vector2i(0, 0), spriteSize, 8);

	sf::Clock clock;
	while (window.isOpen())
	{
		sf::Time deltaTime = clock.restart();

		animator.Update(deltaTime);

		window.clear(sf::Color::Black);
		window.draw(sprite);
		window.display();
	}
}
```

## [<主页](https://www.wangdekui.com/)
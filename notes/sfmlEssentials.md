# SFML Essentials

### [<主页](https://www.wangdekui.com/)

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

		//Render objects

		window.display();
	}

	return 0;
}
```

## [<主页](https://www.wangdekui.com/)
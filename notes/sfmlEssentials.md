# SFML Essentials

### [<主页](https://www.wangdekui.com/)

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

	// Game Loop
	while (window.isOpen())
	{
		/*
			1,Handle input
			2,Update frame
			3,Render frame
		*/
	}
}
```

## [<主页](https://www.wangdekui.com/)
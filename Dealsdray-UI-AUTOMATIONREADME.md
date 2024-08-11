package ui;

import org.openqa.selenium.WebDriver;
import org.openqa.selenium.chrome.ChromeDriver;
import org.openqa.selenium.chrome.ChromeOptions;
import org.openqa.selenium.edge.EdgeDriver;
import org.openqa.selenium.edge.EdgeOptions;
import org.openqa.selenium.Capabilities;
import org.openqa.selenium.Dimension;
import org.openqa.selenium.OutputType;
import org.openqa.selenium.TakesScreenshot;
import org.openqa.selenium.io.FileHandler;
import org.openqa.selenium.remote.RemoteWebDriver;
import org.testng.annotations.AfterTest;
import org.testng.annotations.BeforeTest;
import org.testng.annotations.Parameters;
import org.testng.annotations.Test;

import java.io.File;
import java.io.IOException;
import java.net.MalformedURLException;
import java.net.URL;
import java.util.Arrays;
import java.util.List;
import java.util.concurrent.TimeUnit;

public class UiTesting {

    private WebDriver driver;

    // List of URLs to capture screenshots from
    private final List<String> urls = Arrays.asList(
        "https://www.getcalley.com/",
        "https://www.getcalley.com/calley-lifetime-offer/",
        "https://www.getcalley.com/see-a-demo/",
        "https://www.getcalley.com/calley-teams-features/",
        "https://www.getcalley.com/calley-pro-features/"
    );

    // List of resolutions to capture screenshots at
    private final List<int[]> resolutions = Arrays.asList(
        new int[]{1920, 1080},  // Desktop resolutions
        new int[]{1366, 768},
        new int[]{1536, 864},
        new int[]{360, 640},    // Mobile resolutions
        new int[]{414, 896},
        new int[]{375, 667}
    );

    @Parameters("browser")
    @BeforeTest
    public void setup(String browser) throws MalformedURLException {
        URL gridUrl = new URL("http://192.168.126.60:4444/wd/hub");

        Capabilities capabilities = null;
        switch (browser.toLowerCase()) {
            case "chrome":
                ChromeOptions chromeOptions = new ChromeOptions();
                capabilities = chromeOptions;
                break;
            case "microsoftedge":
                EdgeOptions edgeOptions = new EdgeOptions();
                capabilities = edgeOptions;
                break;
            // Add other browser cases if needed
            default:
                throw new IllegalArgumentException("Browser not supported: " + browser);
        }

        driver = new RemoteWebDriver(gridUrl, capabilities);
        driver.manage().timeouts().implicitlyWait(10, TimeUnit.SECONDS);
        driver.manage().window().maximize();
    }

    @Test
    public void captureScreenshots() {
        // Create the directory if it doesn't exist
        String browserName = ((RemoteWebDriver) driver).getCapabilities().getBrowserName();
        String downloadPath = "C:\\Users\\harih\\Downloads\\dealsdray_" + browserName + "\\";
        File directory = new File(downloadPath);
        if (!directory.exists()) {
            directory.mkdirs();
        }

        // Loop through each URL and each resolution to capture screenshots
        for (String url : urls) {
            driver.get(url);
            for (int[] resolution : resolutions) {
                // Set the window size
                driver.manage().window().setSize(new Dimension(resolution[0], resolution[1]));

                // Generate the filename based on the URL and resolution
                String filename = url.replace("https://", "").replace("/", "_") + "_" + resolution[0] + "x" + resolution[1] + ".png";
                filename = filename.replace("_", "-"); // Replace underscores with hyphens for better readability

                // Take screenshot and save it to the specified folder
                try {
                    File screenshot = ((TakesScreenshot) driver).getScreenshotAs(OutputType.FILE);
                    FileHandler.copy(screenshot, new File(downloadPath + filename));
                } catch (IOException e) {
                    System.err.println("Failed to save screenshot: " + e.getMessage());
                }
            }
        }
    }

    @AfterTest
    public void tearDown() {
        if (driver != null) {
            driver.quit();
        }
    }
}

# Hi there!👋

I'm KurisaW,or you can call me yifang.

```yaml
🔭 Now I am a junior student ...
🌱 I’m currently learning RT-Thread、linux、ROS ...
👯 I’m looking to collaborate on Embedded or neural network ...
🤔 I'm looking forward to sharing my experience and learning to help some beginners get through the rookie phase faster ...
💬 Ask me about Linux、Embedded ...

😊 I will be happy to discuss technology and knowledge with you and look forward to your visit!
```

`Contact me:`

* [Github Address :https://github.com/kurisaW](https://github.com/kurisaW)
* [Email :yifang.wangyq@foxmail.com](mailto:yifang.wangyq@foxmail.com)
* [My Website : https://kurisaw.github.io/](https://kurisaw.github.io/)

---

# How to use the `Hugo Theme Stack Starter Template`?

This is a quick start template for [Hugo theme Stack](https://github.com/CaiJimmy/hugo-theme-stack). It uses [Hugo modules](https://gohugo.io/hugo-modules/) feature to load the theme.

It comes with a basic theme structure and configuration. GitHub action has been set up to deploy the theme to a public GitHub page automatically. Also, there's a cron job to update the theme automatically everyday.

To get started:

1. Click *Use this template*, and create your repository on GitHub.
![image](https://user-images.githubusercontent.com/98592772/210561857-e235865f-76aa-4ca5-a309-3dfce64da916.png)

2. Once the repository is created, create a GitHub codespace asociated with it.
![image](https://user-images.githubusercontent.com/98592772/210561721-27a39816-e194-412e-8ea7-8b490630a136.png)

3. And voila! You're ready to go. The codespace has been configured with the latest version of Hugo extended, just run `hugo server` in the terminal and see your new site in action.

4. Check `config` folder for the configuration files. You can edit them to suit your needs. Make sure to update the `baseurl` property in `config/_default/config.toml` to your site's URL.

5. Once you're done editing the site, just commit it and push it. GitHub action will deploy the site automatically to GitHub page asociated with the repository.
![image](https://user-images.githubusercontent.com/98592772/210562134-0b328d80-c66d-46c8-b128-2514dd769fc7.png)
---

In case you don't want to use GitHub codespace, you can also run this template in your local machine. **You need to install Git, Go and Hugo extended locally.**

### Update theme manually

Run:

```bash
hugo mod get -u github.com/CaiJimmy/hugo-theme-stack/v3
hugo mod tidy
```

> This starter template has been configured with `v3` version of theme. Due to the limitation of Go module, once the `v4` or up version of theme is released, you need to update the theme manually. (Modifying `config/module.toml` file)

### Deploy to another static page hostings

If you want to build this site using another static page hosting, you need to make sure they have Go installed in the machine. 

<details>
  <summary>Vercel</summary>
  
You need to overwrite build command to install manually Go:

```
amazon-linux-extras install golang1.11 && hugo --gc --minify
```

![](https://user-images.githubusercontent.com/5889006/156917172-01e4d418-3469-4ffb-97e4-a905d28b8424.png)

Make sure also to specify Hugo version in the environment variable `HUGO_VERSION` (Use the latest version of Hugo extended):

![Environment variable](https://user-images.githubusercontent.com/5889006/156917212-afb7c70d-ab85-480f-8288-b15781a462c0.png)
</details>

# 模板渲染

`router.go` 的 `func (p *ControllerRegister) ServeHTTP(rw http.ResponseWriter, r *http.Request)` 方法中有：

```go
//render template
if !context.ResponseWriter.Started && context.Output.Status == 0 {
    if BConfig.WebConfig.AutoRender {
	   if err := execController.Render(); err != nil {
			panic(err)
		}
	}
}
```

其中 `execController.Render()` 的实现为（见 `controller.go` 中的 `func (c *Controller) Render() error` 方法） ：

```go
func (c *Controller) Render() error {
	if !c.EnableRender {
		return nil
	}
	rb, err := c.RenderBytes()
	if err != nil {
		return err
	}
	c.Ctx.Output.Header("Content-Type", "text/html; charset=utf-8")
	c.Ctx.Output.Body(rb)
	return nil
}
```

其调用的 `c.RenderBytes()` 实现为：

```go
// RenderBytes returns the bytes of rendered template string. Do not send out response.
func (c *Controller) RenderBytes() ([]byte, error) {
	//if the controller has set layout, then first get the tplname's content set the content to the layout
	var buf bytes.Buffer
	if c.Layout != "" {
		if c.TplName == "" {
			c.TplName = strings.ToLower(c.controllerName) + "/" + strings.ToLower(c.actionName) + "." + c.TplExt
		}

		if BConfig.RunMode == DEV {
			buildFiles := []string{c.TplName}
			if c.LayoutSections != nil {
				for _, sectionTpl := range c.LayoutSections {
					if sectionTpl == "" {
						continue
					}
					buildFiles = append(buildFiles, sectionTpl)
				}
			}
			BuildTemplate(BConfig.WebConfig.ViewsPath, buildFiles...)
		}
		if _, ok := BeeTemplates[c.TplName]; !ok {
			panic("can't find templatefile in the path:" + c.TplName)
		}
		err := BeeTemplates[c.TplName].ExecuteTemplate(&buf, c.TplName, c.Data)
		if err != nil {
			Trace("template Execute err:", err)
			return nil, err
		}
		c.Data["LayoutContent"] = template.HTML(buf.String())

		if c.LayoutSections != nil {
			for sectionName, sectionTpl := range c.LayoutSections {
				if sectionTpl == "" {
					c.Data[sectionName] = ""
					continue
				}

				buf.Reset()
				err = BeeTemplates[sectionTpl].ExecuteTemplate(&buf, sectionTpl, c.Data)
				if err != nil {
					Trace("template Execute err:", err)
					return nil, err
				}
				c.Data[sectionName] = template.HTML(buf.String())
			}
		}

		buf.Reset()
		err = BeeTemplates[c.Layout].ExecuteTemplate(&buf, c.Layout, c.Data)
		if err != nil {
			Trace("template Execute err:", err)
			return nil, err
		}
		return buf.Bytes(), nil
	}

	if c.TplName == "" {
		c.TplName = strings.ToLower(c.controllerName) + "/" + strings.ToLower(c.actionName) + "." + c.TplExt
	}
	if BConfig.RunMode == DEV {
		BuildTemplate(BConfig.WebConfig.ViewsPath, c.TplName)
	}
	if _, ok := BeeTemplates[c.TplName]; !ok {
		panic("can't find templatefile in the path:" + c.TplName)
	}
	buf.Reset()
	err := BeeTemplates[c.TplName].ExecuteTemplate(&buf, c.TplName, c.Data)
	if err != nil {
		Trace("template Execute err:", err)
		return nil, err
	}
	return buf.Bytes(), nil
}
```



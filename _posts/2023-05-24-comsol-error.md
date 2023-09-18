---
title: "Solution to 'SelectionOutOfBoundsException' in COMSOL"
date: 2023-05-24T16:30:00+09:00
category: FEA
tag: Debug
toc: true
header:
  teaser: /assets/images/20230524-error.PNG
---

Recently, I stumbled upon an unusual error while tweaking an optimization model in COMSOL when I am turning a model from a quarter-symmetry to its full geometry. This modification was necessary as I was switching the material model from isotropy to anisotropy.

During the process, an unexpected error occurred. Unlike typical errors that highlight relevant nodes with little red markers, this one provided no such convenience. It gave no further information about its origin within the Graphical User Interface.

<figure style="width: 600px" class="align-center">
  <a href="/assets/images/20230524-error.PNG" alt="An unexpected error occurred.">
  <img src="/assets/images/20230524-error.PNG" alt=""></a>
  <figcaption>The error popup.</figcaption>
</figure>

Upon clicking the button in the popup, the log file revealed an exception termed "SelectionOutOfBoundsException", triggered by an "Illegal_input_vector_illegal_entity_number" error.

```html
2023-05-24T17:47:46.505+0900 ERROR    com.comsol.model.data.SelectionOutOfBoundsException: Illegal_input_vector_illegal_entity_number [com.comsol.widgets]
com.comsol.util.exceptions.BridgeRuntimeException: com.comsol.model.data.SelectionOutOfBoundsException: Illegal_input_vector_illegal_entity_number
	at com.comsol.bridge.BridgeViewManager.showErrorDialog(SourceFile:261)
	at com.comsol.guimph.actions.dj.p(SourceFile:284)
	at com.comsol.guimph.actions.dj.doPerformAction(SourceFile:210)
	at com.comsol.guimph.actions.MphContextAction.doPerformAction(SourceFile:176)
	at com.comsol.gui.c.performAction(SourceFile:142)
	at com.comsol.widgets.actions.CsAction.handleAction(SourceFile:383)
	at com.comsol.widgets.actions.CsAction.triggerActionAndUpdateSelect(SourceFile:515)
	at com.comsol.bridge.command.cp.a(SourceFile:153)
	at com.comsol.bridge.command.cy.run(SourceFile:50)
	at com.comsol.bridge.command.l.c(SourceFile:213)
	at com.comsol.bridge.command.l.a(SourceFile:203)
	at com.comsol.bridge.command.l$1.run(SourceFile:94)
	at com.comsol.util.thread.SuspendableTasks$1.run(SourceFile:111)
	at java.base/java.util.concurrent.ThreadPoolExecutor.runWorker(Unknown Source)
	at java.base/java.util.concurrent.ThreadPoolExecutor$Worker.run(Unknown Source)
	at java.base/java.lang.Thread.run(Unknown Source)
Caused by: com.comsol.model.data.SelectionOutOfBoundsException: Illegal_input_vector_illegal_entity_number
	at com.comsol.model.selections.SelectionMethod.a(SourceFile:1493)
	at com.comsol.model.selections.SelectionMethod.set(SourceFile:938)
	at com.comsol.model.data.primitive.SelectionPrim.set(SourceFile:358)
	at com.comsol.physics.common.au$2.createDefaultPlots(SourceFile:392)
	at com.comsol.model.util.PlotUtil.a(SourceFile:608)
	at com.comsol.model.util.PlotUtil.addDefaultPlot(SourceFile:1294)
	at com.comsol.model.util.PlotUtil.resultSetupNoGui(SourceFile:97)
	at com.comsol.guimph.util.f.a(SourceFile:462)
	at com.comsol.guimph.util.f.a(SourceFile:93)
	at com.comsol.guimph.actions.af.e(SourceFile:66)
	at com.comsol.guimph.actions.fg.doAction(SourceFile:51)
	at com.comsol.guimph.actions.d.a(SourceFile:48)
	at com.comsol.guimph.util.u.a(SourceFile:25)
	at com.comsol.guimph.progress.ProgressView.run(SourceFile:691)
	at com.comsol.bridge.BridgeProgressView.run(SourceFile:184)
	at com.comsol.guimph.actions.dj.p(SourceFile:278)
	... 14 more
```

The log file suggested the issue might stem from incorrect references to specific geometrical features. However, no matter how I rebuilt the geometry or reassigned boundaries and selections, the error persisted.

One notable characteristic of this error was that each time the message appeared, new solutions would show up under the result node, which implies that the problem might be tied to the solver configuration.

Solution: <ins>clear all the previous solutions in the study node</ins> and <ins>reset the solver configuration</ins>.

This particular error seems to stem from some residual settings in the solver during previous runs with different geometries.

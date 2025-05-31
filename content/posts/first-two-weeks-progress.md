+++
date = '2025-05-31T18:00:00+03:00'
draft = false
title = 'GSoC 2025: First Two Weeks Progress Report'
tags = ['GSoC', 'GNOME', 'Papers', 'open-source']
categories = ['GSoC Journey']
+++

The first two weeks of my Google Summer of Code (GSoC) journey with GNOME Papers have been both exciting and productive. I had the opportunity to meet my mentors, discuss the project goals, and dive into my first major task: improving the way document mutex locks are handled in the codebase.

---

## 🤝 Mentor Meeting & Planning

We kicked off with a meeting to get to know each other and to discuss the open [Merge Request 499](https://gitlab.gnome.org/GNOME/Incubator/papers/-/merge_requests/499). The MR focuses on moving document mutex locks from the `libview`/shell layer down to the individual backends (DjVu, PDF, TIFF, Comics). We also outlined the remaining work and clarified how to approach it for the best results.

This initial meeting was crucial for setting expectations and understanding the technical requirements of the project. My mentors provided valuable insights into the GNOME Papers architecture and helped me understand the performance implications of the current mutex implementation.

---

## 🔧 Technical Work: Mutex Lock Refactoring

Previously, the code used `pps_document_doc_mutex_lock/unlock` for document locking. However, this approach caused performance issues, especially when scrolling through documents. The main problem was that the application would become less smooth because the main loop made frequent backend calls, and if another thread was running, it could lock up the entire application.

To address this, I replaced all uses of `pps_document_doc_mutex_lock/unlock` with GLib's `g_rw_lock_writer_lock/unlock` and `g_rw_lock_reader_lock/unlock`. Each backend now manages its own mutex, allowing for more granular and efficient locking.

### Example: DjVu Backend
```c
g_rw_lock_writer_lock (&djvu_document->rwlock);
// ... critical section ...
g_rw_lock_writer_unlock (&djvu_document->rwlock);
```

### Example: TIFF Backend
```c
g_rw_lock_writer_lock (&tiff_document->rwlock);
// ... critical section ...
g_rw_lock_writer_unlock (&tiff_document->rwlock);
```

### Example: PDF Backend
```c
g_rw_lock_writer_lock (&self->rwlock);
retval = poppler_document_save (self->document, uri, &poppler_error);
// ... handle result ...
g_rw_lock_writer_unlock (&self->rwlock);
```

### Example: Comics Backend
```c
g_rw_lock_writer_lock (&comics_document->rwlock);
// ... critical section ...
g_rw_lock_writer_unlock (&comics_document->rwlock);
```

This change allows each backend to handle its own locking logic, improving performance and responsiveness while maintaining thread safety across all document types.

---

## 📈 Impact

The refactoring work has yielded several important benefits:

- **🚀 Performance:** The application is now much smoother, especially during scrolling and heavy backend operations. Users will notice a more responsive interface when navigating through documents.

- **🏗️ Code Quality:** The locking logic is now more modular and easier to maintain. Each backend manages its own mutex, making the code more organized and reducing coupling between components.

- **🤝 Collaboration:** The meeting with my mentors helped clarify the direction and set a solid foundation for the rest of the project. Clear communication and planning are essential for a successful GSoC experience.

---

## 🔮 Next Steps

For the upcoming weeks, my focus will shift to research and prototyping:

- **Research Phase:** Study how projects like glycin/Loupe (and possibly WebKit) handle inter-process communication and data exchange. This research will inform the design of the new architecture.

- **Prototype Development:** Build a small prototype starting with a process that loads a hardcoded file using `pps_document_init` and exposes a simple D-Bus interface (e.g., to render a page or get annotations). Then, try the same approach within a single process, using D-Bus to communicate internally and fetch data like annotations from `libdocument`.

This prototyping phase will be crucial for validating the technical approach and identifying potential challenges early in the development process.

---

## 🙏 Acknowledgments

Thank you to my mentors and the GNOME community for the support so far. The guidance and feedback have been invaluable in helping me understand both the technical aspects and the broader goals of the project.

Looking forward to sharing more progress in the coming weeks! 🚀

---

*This post is part of my GSoC 2025 journey with GNOME. Follow along for more updates on improving GNOME Papers!*
